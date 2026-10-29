# Plugins

Plugin packages, instances, consent and deployment.

Back to the [web API reference](README.md).

## `GET /api/config/plugin-instances`

`GET /api/config/plugin-instances`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Every plugin instance with its deployments. Body: [`PluginInstancesResponse`](schemas.md#plugininstancesresponse) (`application/json`)
  - `node_id` *string*, **required** — Id of the node that answered; its deployments carry its own observations.
  - `instances` *array of [`PluginInstanceView`](schemas.md#plugininstanceview)*, **required** — Every plugin instance in the cluster.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugin-instances",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugin-instances")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Every plugin instance with its deployments.

```json
{
  "node_id": "node-7f3a9c",
  "instances": [
    {
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "plugin_key": "local",
      "alias": "local",
      "display_name": "",
      "is_default": true,
      "created_at": "2026-10-01T08:00:00Z",
      "schema": [
        {
          "key": "root",
          "type": "string"
        }
      ],
      "config": {
        "revision": 5,
        "settings": {
          "multi_destination": "sync_missing",
          "root": "/srv/audiobooks"
        },
        "secret_refs": []
      },
      "deployments": [
        {
          "deployment_id": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
          "node_id": "node-7f3a9c",
          "desired": "present",
          "revision": 3
        }
      ],
      "placement": {
        "replicas": 0,
        "constraints": [
          "target=linux-x64-gnu",
          "!drain"
        ],
        "revision": 2,
        "placed": 1,
        "detail": "",
        "scheduled_at": "2026-10-28T19:07:00Z",
        "updated_at": "2026-10-28T19:05:00Z"
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/plugin-instances` body: every instance as this node\nsees it.",
  "required": [
    "node_id",
    "instances"
  ],
  "properties": {
    "node_id": {
      "type": "string",
      "description": "Id of the node that answered; its deployments carry its own\nobservations.",
      "example": "node-7f3a9c"
    },
    "instances": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One plugin instance with its schema, document, and deployments.",
        "required": [
          "plugin_instance_id",
          "plugin_key",
          "alias",
          "display_name",
          "is_default",
          "created_at",
          "schema",
          "deployments"
        ],
        "properties": {
          "plugin_instance_id": {
            "type": "string",
            "description": "Instance id.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "plugin_key": {
            "type": "string",
            "description": "Canonical plugin key.",
            "example": "local"
          },
          "alias": {
            "type": "string",
            "description": "Manifest id recorded with the pinned schema (display alias).",
            "example": "local"
          },
          "display_name": {
            "type": "string",
            "description": "Operator-facing name; empty for the default instance.",
            "example": ""
          },
          "is_default": {
            "type": "boolean",
            "description": "True when a bare plugin key or alias routes to this instance.",
            "example": true
          },
          "created_at": {
            "type": "string",
            "description": "RFC 3339 creation time.",
            "example": "2026-10-01T08:00:00Z"
          },
          "schema": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One setting a plugin declares in its `plugin.toml` `[[settings]]`:\nits key, value type, default and help text.",
              "required": [
                "key",
                "type"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Setting key (`[a-z][a-z0-9_]*`, at most 64 bytes).",
                  "example": "root"
                },
                "label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Operator-facing label; omitted when the key is shown.",
                  "example": "Library root"
                },
                "type": {
                  "type": "string",
                  "description": "Value type: `string`, `bool`, `integer`, `enum` or `secret`.",
                  "enum": [
                    "string",
                    "bool",
                    "integer",
                    "enum",
                    "secret"
                  ],
                  "example": "string"
                },
                "values": {
                  "type": [
                    "array",
                    "null"
                  ],
                  "items": {
                    "type": "object",
                    "description": "One allowed value of an `enum` setting: the stored id and its label.",
                    "required": [
                      "id"
                    ],
                    "properties": {
                      "id": {
                        "type": "string",
                        "description": "Stored value.",
                        "example": "sync_missing"
                      },
                      "label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Operator-facing label; omitted when the id is shown.",
                        "example": "Sync missing"
                      }
                    }
                  },
                  "description": "Allowed values of an `enum` setting; omitted for other kinds."
                },
                "default": {
                  "description": "Default applied when the instance document has no value; omitted when there is none. A string, boolean or integer, as the setting's `type` says.",
                  "examples": [
                    "/srv/audiobooks"
                  ]
                },
                "required": {
                  "type": [
                    "boolean",
                    "null"
                  ],
                  "description": "True when the instance cannot run without a value; omitted when\nfalse.",
                  "example": true
                },
                "description": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Help text shown beside the setting; omitted when there is none.",
                  "example": "Directory the finished audiobooks are written to."
                }
              }
            },
            "description": "Declared settings from the pinned schema."
          },
          "config": {
            "oneOf": [
              {
                "type": "object",
                "description": "Current document, when the instance has one.",
                "required": [
                  "revision",
                  "settings",
                  "secret_refs"
                ],
                "properties": {
                  "revision": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Compare-and-swap revision.",
                    "example": 5
                  },
                  "settings": {
                    "type": "object",
                    "description": "Scalar settings the operator wrote.",
                    "additionalProperties": {
                      "oneOf": [
                        {
                          "type": "boolean",
                          "description": "JSON boolean."
                        },
                        {
                          "type": "number",
                          "description": "Finite JSON number: an integer kind stores an `i64`, so the schema\nis an unformatted `number` rather than a `double`.",
                          "examples": [
                            8
                          ]
                        },
                        {
                          "type": "string",
                          "description": "JSON string, at most 1024 bytes."
                        }
                      ],
                      "description": "One scalar setting. Nested objects and arrays are not representable.",
                      "example": "/srv/audiobooks"
                    },
                    "propertyNames": {
                      "type": "string"
                    },
                    "example": {
                      "multi_destination": "sync_missing",
                      "root": "/srv/audiobooks"
                    }
                  },
                  "secret_refs": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "description": "A name in `encrypted_secrets`. The ciphertext is not part of the document.",
                      "required": [
                        "key",
                        "name"
                      ],
                      "properties": {
                        "key": {
                          "type": "string",
                          "description": "Key in the `SECRETS` object delivered to the guest.",
                          "example": "api_token"
                        },
                        "name": {
                          "type": "string",
                          "description": "`encrypted_secrets.name` for this instance.",
                          "example": "api_token"
                        }
                      }
                    },
                    "description": "Secret refs; ciphertext stays in `encrypted_secrets`."
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          },
          "deployments": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "This node's view of one deployment row, with its latest observation.",
              "required": [
                "deployment_id",
                "node_id",
                "desired",
                "revision"
              ],
              "properties": {
                "deployment_id": {
                  "type": "string",
                  "description": "Deployment row id.",
                  "example": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f"
                },
                "node_id": {
                  "type": "string",
                  "description": "Node the row targets.",
                  "example": "node-7f3a9c"
                },
                "desired": {
                  "type": "string",
                  "description": "What the row asks the node to do: `present` / `absent` from an\noperator, `placed` / `unplaced` from the scheduler.",
                  "enum": [
                    "present",
                    "absent",
                    "placed",
                    "unplaced"
                  ]
                },
                "revision": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Compare-and-swap revision of the desired row.",
                  "example": 3
                },
                "status": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Latest observation status from that node, when it reported one.",
                  "example": "healthy"
                },
                "detail": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Observation detail (error text, redacted).",
                  "example": ""
                },
                "applied_config_revision": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Document revision the running guest applied.",
                  "example": 5
                },
                "observed_at": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "RFC 3339 time of the observation.",
                  "example": "2026-10-28T19:07:00Z"
                }
              }
            },
            "description": "Deployment rows across nodes."
          },
          "placement": {
            "oneOf": [
              {
                "type": "object",
                "description": "Scheduler placement; absent when the instance is hand-managed.",
                "required": [
                  "replicas",
                  "constraints",
                  "revision",
                  "placed",
                  "detail",
                  "scheduled_at",
                  "updated_at"
                ],
                "properties": {
                  "replicas": {
                    "oneOf": [
                      {
                        "type": "integer",
                        "minimum": 0
                      },
                      {
                        "type": "string",
                        "enum": [
                          "all"
                        ]
                      }
                    ],
                    "description": "A replica count (0 stops every replica) or `\"all\"`."
                  },
                  "constraints": {
                    "type": "array",
                    "items": {
                      "type": "string"
                    },
                    "description": "Label constraints every chosen node must satisfy.",
                    "example": [
                      "target=linux-x64-gnu",
                      "!drain"
                    ]
                  },
                  "revision": {
                    "type": "integer",
                    "format": "int64",
                    "description": "Compare-and-swap revision of the intent.",
                    "example": 2
                  },
                  "placed": {
                    "type": "integer",
                    "format": "int32",
                    "description": "Replicas the last scheduler pass placed.",
                    "example": 1,
                    "minimum": 0
                  },
                  "detail": {
                    "type": "string",
                    "description": "Why fewer than requested are placed; empty when satisfied.",
                    "example": ""
                  },
                  "scheduled_at": {
                    "type": "string",
                    "description": "RFC 3339 time the scheduler last changed the status.",
                    "example": "2026-10-28T19:07:00Z"
                  },
                  "updated_at": {
                    "type": "string",
                    "description": "RFC 3339 time of the last intent write.",
                    "example": "2026-10-28T19:05:00Z"
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
      "description": "Every plugin instance in the cluster."
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

Status `500`: The configuration could not be read.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/plugin-instances`

`POST /api/config/plugin-instances`: a second (non-default) instance of
a plugin, with an empty document so schema defaults apply. With
`node_id`, the instance is local to that node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CreatePluginInstance`](schemas.md#createplugininstance)

- `plugin_key` *string*, **required** — Canonical plugin key; the plugin must have a pinned schema.
- `display_name` *string* — Operator-facing name.
- `node_id` *string or null* — Make the instance local to this node (H-2): it deploys only there and is never placed by the scheduler.

### Responses

- `200` — The new instance. Body: [`PluginInstanceCreated`](schemas.md#plugininstancecreated) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Id of the new instance.
  - `plugin_key` *string*, **required** — Canonical plugin key the instance belongs to.
  - `display_name` *string*, **required** — Operator-facing name, trimmed.
  - `owner_node_id` *string or null* — Node the instance is local to; `null` for a cluster-wide instance.
- `400` — The body is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin, or it has no pinned schema. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The instance id is taken. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"plugin_key": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"plugin_key": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugin-instances",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"plugin_key": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"plugin_key": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugin-instances")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"plugin_key": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new instance.

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "plugin_key": "s3",
  "display_name": "Archive bucket",
  "owner_node_id": "node-7f3a9c"
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/config/plugin-instances` body: the instance that was created.",
  "required": [
    "plugin_instance_id",
    "plugin_key",
    "display_name"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Id of the new instance.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical plugin key the instance belongs to.",
      "example": "s3"
    },
    "display_name": {
      "type": "string",
      "description": "Operator-facing name, trimmed.",
      "example": "Archive bucket"
    },
    "owner_node_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Node the instance is local to; `null` for a cluster-wide instance.",
      "example": "node-7f3a9c"
    }
  }
}
```

Status `400`: The body is invalid.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such plugin, or it has no pinned schema.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: The instance id is taken.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/config/plugin-instances/{id}`

`DELETE /api/config/plugin-instances/{id}`: removes an instance that has
no deployment left.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance was removed. Body: [`PluginInstanceDeleted`](schemas.md#plugininstancedeleted) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Id of the instance that was removed.
  - `deleted` *boolean*, **required** — Always true: removing an instance that is already gone answers the same way.
- `400` — The id is not a plugin instance id. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The instance is a default or still deployed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugin-instances/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance was removed.

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "deleted": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "`DELETE /api/config/plugin-instances/{id}` body.",
  "required": [
    "plugin_instance_id",
    "deleted"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Id of the instance that was removed.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "deleted": {
      "type": "boolean",
      "description": "Always true: removing an instance that is already gone answers the\nsame way.",
      "example": true
    }
  }
}
```

Status `400`: The id is not a plugin instance id.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: The instance is a default or still deployed.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/plugin-instances/{id}/config`

`GET /api/config/plugin-instances/{id}/config`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance's settings document (secrets as names only). Body: [`PluginInstanceConfigResponse`](schemas.md#plugininstanceconfigresponse) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Instance id.
  - `plugin_key` *string*, **required** — Canonical plugin key.
  - `revision` *integer (int64)*, **required** — Compare-and-swap revision of the document.
  - `settings` *map of string to [`SettingValue`](schemas.md#settingvalue)*, **required** — Scalar settings the operator wrote.
  - `secret_refs` *array of [`InstanceSecretRefV1`](schemas.md#instancesecretrefv1)*, **required** — Secret refs; ciphertext stays in `encrypted_secrets`.
  - `effective` *object*, **required** — Settings as the guest sees them: schema defaults overlaid with the stored values. Empty when the plugin has no pinned schema.
  - `schema` *array of [`SettingSpec`](schemas.md#settingspec)*, **required** — Declared settings from the pinned schema; empty when there is none.
- `400` — The id is not a plugin instance id. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance, or it has no document yet. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/config",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance's settings document (secrets as names only).

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "plugin_key": "local",
  "revision": 5,
  "settings": {
    "root": "/srv/audiobooks"
  },
  "secret_refs": [
    {
      "key": "api_token",
      "name": "api_token"
    }
  ],
  "effective": {},
  "schema": [
    {
      "key": "root",
      "label": "Library root",
      "type": "string",
      "values": [
        {
          "id": "sync_missing"
        }
      ],
      "default": "/srv/audiobooks",
      "required": true,
      "description": "Directory the finished audiobooks are written to."
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/config/plugin-instances/{id}/config` body: the stored document\nand the configuration the guest receives.",
  "required": [
    "plugin_instance_id",
    "plugin_key",
    "revision",
    "settings",
    "secret_refs",
    "effective",
    "schema"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Instance id.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical plugin key.",
      "example": "local"
    },
    "revision": {
      "type": "integer",
      "format": "int64",
      "description": "Compare-and-swap revision of the document.",
      "example": 5
    },
    "settings": {
      "type": "object",
      "description": "Scalar settings the operator wrote.",
      "additionalProperties": {
        "oneOf": [
          {
            "type": "boolean",
            "description": "JSON boolean."
          },
          {
            "type": "number",
            "description": "Finite JSON number: an integer kind stores an `i64`, so the schema\nis an unformatted `number` rather than a `double`.",
            "examples": [
              8
            ]
          },
          {
            "type": "string",
            "description": "JSON string, at most 1024 bytes."
          }
        ],
        "description": "One scalar setting. Nested objects and arrays are not representable.",
        "example": "/srv/audiobooks"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "root": "/srv/audiobooks"
      }
    },
    "secret_refs": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A name in `encrypted_secrets`. The ciphertext is not part of the document.",
        "required": [
          "key",
          "name"
        ],
        "properties": {
          "key": {
            "type": "string",
            "description": "Key in the `SECRETS` object delivered to the guest.",
            "example": "api_token"
          },
          "name": {
            "type": "string",
            "description": "`encrypted_secrets.name` for this instance.",
            "example": "api_token"
          }
        }
      },
      "description": "Secret refs; ciphertext stays in `encrypted_secrets`."
    },
    "effective": {
      "type": "object",
      "description": "Settings as the guest sees them: schema defaults overlaid with the\nstored values. Empty when the plugin has no pinned schema."
    },
    "schema": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One setting a plugin declares in its `plugin.toml` `[[settings]]`:\nits key, value type, default and help text.",
        "required": [
          "key",
          "type"
        ],
        "properties": {
          "key": {
            "type": "string",
            "description": "Setting key (`[a-z][a-z0-9_]*`, at most 64 bytes).",
            "example": "root"
          },
          "label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing label; omitted when the key is shown.",
            "example": "Library root"
          },
          "type": {
            "type": "string",
            "description": "Value type: `string`, `bool`, `integer`, `enum` or `secret`.",
            "enum": [
              "string",
              "bool",
              "integer",
              "enum",
              "secret"
            ],
            "example": "string"
          },
          "values": {
            "type": [
              "array",
              "null"
            ],
            "items": {
              "type": "object",
              "description": "One allowed value of an `enum` setting: the stored id and its label.",
              "required": [
                "id"
              ],
              "properties": {
                "id": {
                  "type": "string",
                  "description": "Stored value.",
                  "example": "sync_missing"
                },
                "label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Operator-facing label; omitted when the id is shown.",
                  "example": "Sync missing"
                }
              }
            },
            "description": "Allowed values of an `enum` setting; omitted for other kinds."
          },
          "default": {
            "description": "Default applied when the instance document has no value; omitted when there is none. A string, boolean or integer, as the setting's `type` says.",
            "examples": [
              "/srv/audiobooks"
            ]
          },
          "required": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "True when the instance cannot run without a value; omitted when\nfalse.",
            "example": true
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Help text shown beside the setting; omitted when there is none.",
            "example": "Directory the finished audiobooks are written to."
          }
        }
      },
      "description": "Declared settings from the pinned schema; empty when there is none."
    }
  }
}
```

Status `400`: The id is not a plugin instance id.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance, or it has no document yet.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `500`: The configuration could not be read.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/plugin-instances/{id}/config`

`PUT /api/config/plugin-instances/{id}/config`.

Operator principal only. The route is mounted on the operator router.
The document is validated against the plugin's pinned schema, so this
node does not need the guest installed.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Request body

`application/json`: [`PutPluginInstanceConfig`](schemas.md#putplugininstanceconfig)

- `expected_revision` *integer (int64)*, **required** — Revision the caller last observed.
- `settings` *object* — Scalar settings. Secret values are not accepted here.
- `secret_refs` *array of object* — Secret refs to keep. Omitted refs are dropped from the document; the ciphertext stays sealed until the secret is overwritten.
- `secrets` *map of string to string* — Secret values to seal, keyed by the declared `secret` setting. Each is sealed into `encrypted_secrets` and referenced from the document; the plaintext is never stored in the document or echoed back.

### Responses

- `200` — The document after the change. Body: [`PluginInstanceConfigChanged`](schemas.md#plugininstanceconfigchanged) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Instance id.
  - `revision` *integer (int64)*, **required** — Revision after the write; unchanged when the same write was replayed.
  - `settings` *map of string to [`SettingValue`](schemas.md#settingvalue)*, **required** — Scalar settings now stored.
  - `secret_refs` *array of [`InstanceSecretRefV1`](schemas.md#instancesecretrefv1)*, **required** — Secret refs now stored, the newly sealed secrets included.
- `400` — The body is invalid, or a value does not fit the pinned schema. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance, or its plugin has no pinned schema. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — `expected_revision` is not the current revision. Body: [`RevisionConflict`](schemas.md#revisionconflict) (`application/json`)
  - `error` *string*, **required** — Always `revision_conflict`.
  - `current_revision` *integer (int64)*, **required** — The revision to read and retry against.
- `500` — The configuration could not be read or written, or a secret was not stored. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 0}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/config",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The document after the change.

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "revision": 6,
  "settings": {
    "root": "/srv/audiobooks"
  },
  "secret_refs": [
    {
      "key": "api_token",
      "name": "api_token"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/plugin-instances/{id}/config` body: the document after\nthe write.",
  "required": [
    "plugin_instance_id",
    "revision",
    "settings",
    "secret_refs"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Instance id.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "revision": {
      "type": "integer",
      "format": "int64",
      "description": "Revision after the write; unchanged when the same write was\nreplayed.",
      "example": 6
    },
    "settings": {
      "type": "object",
      "description": "Scalar settings now stored.",
      "additionalProperties": {
        "oneOf": [
          {
            "type": "boolean",
            "description": "JSON boolean."
          },
          {
            "type": "number",
            "description": "Finite JSON number: an integer kind stores an `i64`, so the schema\nis an unformatted `number` rather than a `double`.",
            "examples": [
              8
            ]
          },
          {
            "type": "string",
            "description": "JSON string, at most 1024 bytes."
          }
        ],
        "description": "One scalar setting. Nested objects and arrays are not representable.",
        "example": "/srv/audiobooks"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "root": "/srv/audiobooks"
      }
    },
    "secret_refs": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A name in `encrypted_secrets`. The ciphertext is not part of the document.",
        "required": [
          "key",
          "name"
        ],
        "properties": {
          "key": {
            "type": "string",
            "description": "Key in the `SECRETS` object delivered to the guest.",
            "example": "api_token"
          },
          "name": {
            "type": "string",
            "description": "`encrypted_secrets.name` for this instance.",
            "example": "api_token"
          }
        }
      },
      "description": "Secret refs now stored, the newly sealed secrets included."
    }
  }
}
```

Status `400`: The body is invalid, or a value does not fit the pinned schema.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance, or its plugin has no pinned schema.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: `expected_revision` is not the current revision.

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

Status `500`: The configuration could not be read or written, or a secret was not stored.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
Deployed switch. Creates the row when the node has none.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.
- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutPluginDeployment`](schemas.md#putplugindeployment)

- `desired` *[`DesiredState`](schemas.md#desiredstate)*, **required** — What the node should do: `present` or `absent` (the scheduler's `placed` / `unplaced` are accepted too).
- `expected_revision` *integer (int64) or null* — Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current".

### Responses

- `200` — The deployment after the change. Body: [`PluginDeploymentChanged`](schemas.md#plugindeploymentchanged) (`application/json`)
  - `deployment_id` *string*, **required** — Deployment row id.
  - `plugin_instance_id` *string*, **required** — Instance the row belongs to.
  - `node_id` *string*, **required** — Node the row targets.
  - `desired` *[`DesiredState`](schemas.md#desiredstate)*, **required** — Desired state now stored.
  - `revision` *integer (int64)*, **required** — Compare-and-swap revision after the write.
- `400` — The body is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance or node. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The instance is placed by the scheduler (an `error` message), or `expected_revision` is stale (`revision_conflict` with `current_revision`). Body: [`DeploymentConflict`](schemas.md#deploymentconflict) (`application/json`)
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"desired": "present"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"desired": "present"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"desired": "present"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"desired": "present"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"desired": "present"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The deployment after the change.

```json
{
  "deployment_id": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "node_id": "node-7f3a9c",
  "desired": "present",
  "revision": 4
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` body: the\ndeployment row after the write.",
  "required": [
    "deployment_id",
    "plugin_instance_id",
    "node_id",
    "desired",
    "revision"
  ],
  "properties": {
    "deployment_id": {
      "type": "string",
      "description": "Deployment row id.",
      "example": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f"
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "Instance the row belongs to.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "node_id": {
      "type": "string",
      "description": "Node the row targets.",
      "example": "node-7f3a9c"
    },
    "desired": {
      "type": "string",
      "description": "Desired state now stored.",
      "enum": [
        "present",
        "absent",
        "placed",
        "unplaced"
      ]
    },
    "revision": {
      "type": "integer",
      "format": "int64",
      "description": "Compare-and-swap revision after the write.",
      "example": 4
    }
  }
}
```

Status `400`: The body is invalid.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance or node.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: The instance is placed by the scheduler (an `error` message), or `expected_revision` is stale (`revision_conflict` with `current_revision`).

```json
{
  "error": "revision_conflict",
  "current_revision": 13
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "`expected_revision` lost to a newer revision of the row.",
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
    },
    {
      "type": "object",
      "description": "The scheduler places the instance, so its deployments are not hand-managed; `error` says so and names the way out.",
      "required": [
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
  ],
  "description": "`PUT /api/config/plugin-instances/{id}/deployments/{node_id}` `409` body: the two conflicts a deployment write can hit."
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/plugin-instances/{id}/placement`

`PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
deployments to the scheduler with a replica count and constraints.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Request body

`application/json`: [`PutPluginPlacement`](schemas.md#putpluginplacement)

- `replicas` *integer or `all`*, **required** — A replica count (0 stops every replica) or `"all"`.
- `constraints` *array of string* — Label constraints (`name`, `name=value`, `!name`).
- `expected_revision` *integer (int64) or null* — Current revision to replace; omit (or 0) to create.

### Responses

- `200` — The scheduler now places the instance. Body: [`PluginPlacementChanged`](schemas.md#pluginplacementchanged) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Instance the scheduler now places.
  - `placement` *[`PlacementView`](schemas.md#placementview)*, **required** — The stored intent and the last scheduler pass's result.
- `400` — The body is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — `expected_revision` is not the current revision. Body: [`RevisionConflict`](schemas.md#revisionconflict) (`application/json`)
  - `error` *string*, **required** — Always `revision_conflict`.
  - `current_revision` *integer (int64)*, **required** — The revision to read and retry against.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"replicas": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"replicas": 0}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"replicas": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"replicas": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"replicas": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The scheduler now places the instance.

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "placement": {
    "replicas": 0,
    "constraints": [
      "target=linux-x64-gnu",
      "!drain"
    ],
    "revision": 2,
    "placed": 1,
    "detail": "",
    "scheduled_at": "2026-10-28T19:07:00Z",
    "updated_at": "2026-10-28T19:05:00Z"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/plugin-instances/{id}/placement` body: the placement\nafter the write.",
  "required": [
    "plugin_instance_id",
    "placement"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Instance the scheduler now places.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "placement": {
      "type": "object",
      "description": "The stored intent and the last scheduler pass's result.",
      "required": [
        "replicas",
        "constraints",
        "revision",
        "placed",
        "detail",
        "scheduled_at",
        "updated_at"
      ],
      "properties": {
        "replicas": {
          "oneOf": [
            {
              "type": "integer",
              "minimum": 0
            },
            {
              "type": "string",
              "enum": [
                "all"
              ]
            }
          ],
          "description": "A replica count (0 stops every replica) or `\"all\"`."
        },
        "constraints": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Label constraints every chosen node must satisfy.",
          "example": [
            "target=linux-x64-gnu",
            "!drain"
          ]
        },
        "revision": {
          "type": "integer",
          "format": "int64",
          "description": "Compare-and-swap revision of the intent.",
          "example": 2
        },
        "placed": {
          "type": "integer",
          "format": "int32",
          "description": "Replicas the last scheduler pass placed.",
          "example": 1,
          "minimum": 0
        },
        "detail": {
          "type": "string",
          "description": "Why fewer than requested are placed; empty when satisfied.",
          "example": ""
        },
        "scheduled_at": {
          "type": "string",
          "description": "RFC 3339 time the scheduler last changed the status.",
          "example": "2026-10-28T19:07:00Z"
        },
        "updated_at": {
          "type": "string",
          "description": "RFC 3339 time of the last intent write.",
          "example": "2026-10-28T19:05:00Z"
        }
      }
    }
  }
}
```

Status `400`: The body is invalid.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: `expected_revision` is not the current revision.

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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/config/plugin-instances/{id}/placement`

`DELETE /api/config/plugin-instances/{id}/placement`: return the
instance to hand-managed deployments; its deployments stay as they
are.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance is back to manual deployment. Body: [`PluginPlacementDeleted`](schemas.md#pluginplacementdeleted) (`application/json`)
  - `plugin_instance_id` *string*, **required** — Instance that is hand-managed again.
  - `deleted` *boolean*, **required** — True when a placement existed and was removed; false when the instance was hand-managed already.
- `400` — The id is not a plugin instance id. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance is back to manual deployment.

```json
{
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "deleted": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "`DELETE /api/config/plugin-instances/{id}/placement` body.",
  "required": [
    "plugin_instance_id",
    "deleted"
  ],
  "properties": {
    "plugin_instance_id": {
      "type": "string",
      "description": "Instance that is hand-managed again.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "deleted": {
      "type": "boolean",
      "description": "True when a placement existed and was removed; false when the\ninstance was hand-managed already.",
      "example": true
    }
  }
}
```

Status `400`: The id is not a plugin instance id.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such instance.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/plugins`

`GET /api/config/plugins`: every installed plugin.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Every installed plugin. Body: [`InstalledPluginsResponse`](schemas.md#installedpluginsresponse) (`application/json`)
  - `plugins` *array of [`InstalledPluginSummary`](schemas.md#installedpluginsummary)*, **required** — Every plugin with at least one installed version, by key.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugins",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugins")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Every installed plugin.

```json
{
  "plugins": [
    {
      "plugin_key": "libro",
      "id": "libro",
      "active_version": "0.4.2",
      "revision": 3,
      "versions": [
        {
          "plugin_key": "libro",
          "id": "libro",
          "version": "0.4.2",
          "active": true,
          "approved": true,
          "publisher_verified": true,
          "coordinate": "registry:plugins.example#source-libro@0.4.2",
          "installed_at": "2026-10-20T10:15:00Z",
          "installed_by": "operator",
          "artifacts": []
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
  "description": "`GET /api/config/plugins` body.",
  "required": [
    "plugins"
  ],
  "properties": {
    "plugins": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One installed plugin: its active version and every installed version,\nnewest first.",
        "required": [
          "plugin_key",
          "versions"
        ],
        "properties": {
          "plugin_key": {
            "type": "string",
            "description": "Canonical plugin key.",
            "example": "libro"
          },
          "id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Manifest id (display alias) of the active version, or of the newest\none when none is active; `null` without versions.",
            "example": "libro"
          },
          "active_version": {
            "type": [
              "string",
              "null"
            ],
            "description": "Active version; `null` when the plugin is installed but not\nactivated.",
            "example": "0.4.2"
          },
          "revision": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Compare-and-swap revision of the active-version pointer; `null`\nwhen nothing is active.",
            "example": 3
          },
          "versions": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One installed version of a plugin, as the API renders it.",
              "required": [
                "plugin_key",
                "id",
                "version",
                "active",
                "approved",
                "publisher_verified",
                "coordinate",
                "installed_at",
                "installed_by",
                "artifacts"
              ],
              "properties": {
                "plugin_key": {
                  "type": "string",
                  "description": "Canonical plugin key.",
                  "example": "libro"
                },
                "id": {
                  "type": "string",
                  "description": "Manifest id (display alias).",
                  "example": "libro"
                },
                "version": {
                  "type": "string",
                  "description": "Semver package version.",
                  "example": "0.4.2"
                },
                "active": {
                  "type": "boolean",
                  "description": "True when this is the plugin's active version.",
                  "example": true
                },
                "approved": {
                  "type": "boolean",
                  "description": "True once an operator approved the version's consent request.",
                  "example": true
                },
                "publisher_verified": {
                  "type": "boolean",
                  "description": "True when the publisher was verified at install time.",
                  "example": true
                },
                "coordinate": {
                  "type": "string",
                  "description": "Package coordinate the version was resolved from; empty for a local\narchive.",
                  "example": "registry:plugins.example#source-libro@0.4.2"
                },
                "installed_at": {
                  "type": "string",
                  "description": "RFC 3339 install time.",
                  "example": "2026-10-20T10:15:00Z"
                },
                "installed_by": {
                  "type": "string",
                  "description": "Audit identity of the installer.",
                  "example": "operator"
                },
                "host_payload_root_sha256": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Payload-root digest of the artifact this host would run; `null` when\nno artifact fits the host's target.",
                  "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
                },
                "artifacts": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One downloadable artifact of an installed version, as the API renders\nit.",
                    "required": [
                      "target",
                      "url",
                      "archive_sha256",
                      "payload_root_sha256"
                    ],
                    "properties": {
                      "target": {
                        "type": "string",
                        "description": "Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).",
                        "example": "linux-x64-gnu"
                      },
                      "url": {
                        "type": "string",
                        "description": "Where nodes fetch the archive: `https:`, `file:`, or\n`blob:sha256:<hex>`.",
                        "example": "https://plugins.example/source-libro-0.4.2-linux-x64-gnu.tar.gz"
                      },
                      "archive_sha256": {
                        "type": "string",
                        "description": "Lowercase hex SHA-256 of the archive bytes.",
                        "example": "9b74c9897bac770ffc029102a200c5de3f4b2b1a0b5e6c7d8e9f0a1b2c3d4e5f"
                      },
                      "payload_root_sha256": {
                        "type": "string",
                        "description": "Payload-root digest of the extracted plugin tree.",
                        "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
                      }
                    }
                  },
                  "description": "Per-target artifacts."
                }
              }
            },
            "description": "Every installed version, newest first."
          }
        }
      },
      "description": "Every plugin with at least one installed version, by key."
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

Status `500`: The configuration could not be read.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/plugins`

`POST /api/config/plugins`: install (and maybe activate) a version.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PostPluginInstall`](schemas.md#postplugininstall)

- `coordinate` *string or null* — Registry coordinate (`cargo:…@v`, `registry:…#name@v`).
- `package` *string or null* — Operator-placed package directory under `plugin-packages/`.
- `activate` *boolean* — Make the new version active even when another is.
- `approve` *boolean* — Record the requested consent as approved.
- `allow_unverified_publisher` *boolean* — Accept a publisher without independent authenticity.

### Responses

- `200` — The installed version. Body: [`PluginInstalled`](schemas.md#plugininstalled) (`application/json`)
  - `created` *boolean*, **required** — True when the version row was written now; false when the same version with the same artifacts was installed already.
  - `plugin_instance_id` *string*, **required** — Id of the plugin's default instance.
  - `version` *[`PluginVersionSummary`](schemas.md#pluginversionsummary)*, **required** — The installed version.
- `400` — The body is invalid: neither or both of `coordinate` and `package`, a malformed coordinate or package name, or a manifest the host rejects. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The publisher is unverified and `allow_unverified_publisher` is off. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `404` — A local package is missing its `package.json` or `archive.tar.gz`. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The version is already installed with different artifacts. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The package catalog could not be reached or answered badly. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The install needs a helper that is unavailable. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugins",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugins")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The installed version.

```json
{
  "created": true,
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "version": {
    "plugin_key": "libro",
    "id": "libro",
    "version": "0.4.2",
    "active": true,
    "approved": true,
    "publisher_verified": true,
    "coordinate": "registry:plugins.example#source-libro@0.4.2",
    "installed_at": "2026-10-20T10:15:00Z",
    "installed_by": "operator",
    "host_payload_root_sha256": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5",
    "artifacts": [
      {
        "target": "linux-x64-gnu",
        "url": "https://plugins.example/source-libro-0.4.2-linux-x64-gnu.tar.gz",
        "archive_sha256": "9b74c9897bac770ffc029102a200c5de3f4b2b1a0b5e6c7d8e9f0a1b2c3d4e5f",
        "payload_root_sha256": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/config/plugins` body: what the install did.",
  "required": [
    "created",
    "plugin_instance_id",
    "version"
  ],
  "properties": {
    "created": {
      "type": "boolean",
      "description": "True when the version row was written now; false when the same\nversion with the same artifacts was installed already.",
      "example": true
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "Id of the plugin's default instance.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "version": {
      "type": "object",
      "description": "The installed version.",
      "required": [
        "plugin_key",
        "id",
        "version",
        "active",
        "approved",
        "publisher_verified",
        "coordinate",
        "installed_at",
        "installed_by",
        "artifacts"
      ],
      "properties": {
        "plugin_key": {
          "type": "string",
          "description": "Canonical plugin key.",
          "example": "libro"
        },
        "id": {
          "type": "string",
          "description": "Manifest id (display alias).",
          "example": "libro"
        },
        "version": {
          "type": "string",
          "description": "Semver package version.",
          "example": "0.4.2"
        },
        "active": {
          "type": "boolean",
          "description": "True when this is the plugin's active version.",
          "example": true
        },
        "approved": {
          "type": "boolean",
          "description": "True once an operator approved the version's consent request.",
          "example": true
        },
        "publisher_verified": {
          "type": "boolean",
          "description": "True when the publisher was verified at install time.",
          "example": true
        },
        "coordinate": {
          "type": "string",
          "description": "Package coordinate the version was resolved from; empty for a local\narchive.",
          "example": "registry:plugins.example#source-libro@0.4.2"
        },
        "installed_at": {
          "type": "string",
          "description": "RFC 3339 install time.",
          "example": "2026-10-20T10:15:00Z"
        },
        "installed_by": {
          "type": "string",
          "description": "Audit identity of the installer.",
          "example": "operator"
        },
        "host_payload_root_sha256": {
          "type": [
            "string",
            "null"
          ],
          "description": "Payload-root digest of the artifact this host would run; `null` when\nno artifact fits the host's target.",
          "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
        },
        "artifacts": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One downloadable artifact of an installed version, as the API renders\nit.",
            "required": [
              "target",
              "url",
              "archive_sha256",
              "payload_root_sha256"
            ],
            "properties": {
              "target": {
                "type": "string",
                "description": "Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).",
                "example": "linux-x64-gnu"
              },
              "url": {
                "type": "string",
                "description": "Where nodes fetch the archive: `https:`, `file:`, or\n`blob:sha256:<hex>`.",
                "example": "https://plugins.example/source-libro-0.4.2-linux-x64-gnu.tar.gz"
              },
              "archive_sha256": {
                "type": "string",
                "description": "Lowercase hex SHA-256 of the archive bytes.",
                "example": "9b74c9897bac770ffc029102a200c5de3f4b2b1a0b5e6c7d8e9f0a1b2c3d4e5f"
              },
              "payload_root_sha256": {
                "type": "string",
                "description": "Payload-root digest of the extracted plugin tree.",
                "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
              }
            }
          },
          "description": "Per-target artifacts."
        }
      }
    }
  }
}
```

Status `400`: The body is invalid: neither or both of `coordinate` and `package`, a malformed coordinate or package name, or a manifest the host rejects.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `403`: The publisher is unverified and `allow_unverified_publisher` is off. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: A local package is missing its `package.json` or `archive.tar.gz`.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: The version is already installed with different artifacts.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `502`: The package catalog could not be reached or answered badly.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `503`: The install needs a helper that is unavailable.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/config/plugins/{key}`

`GET /api/config/plugins/{key}`: one installed plugin with its pinned
settings schema.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Responses

- `200` — The plugin with its installed versions and pinned schema. Body: [`InstalledPluginDetail`](schemas.md#installedplugindetail) (`application/json`)
  - `schema` *[`PinnedSchemaSummary`](schemas.md#pinnedschemasummary) or null* — The pinned settings schema; `null` until a node pins one.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugins/<key>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugins/<key>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The plugin with its installed versions and pinned schema.

```json
{
  "plugin_key": "libro",
  "id": "libro",
  "active_version": "0.4.2",
  "revision": 3,
  "versions": [
    {
      "plugin_key": "libro",
      "id": "libro",
      "version": "0.4.2",
      "active": true,
      "approved": true,
      "publisher_verified": true,
      "coordinate": "registry:plugins.example#source-libro@0.4.2",
      "installed_at": "2026-10-20T10:15:00Z",
      "installed_by": "operator",
      "host_payload_root_sha256": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5",
      "artifacts": [
        {
          "target": "linux-x64-gnu",
          "url": "https://plugins.example/source-libro-0.4.2-linux-x64-gnu.tar.gz",
          "archive_sha256": "9b74c9897bac770ffc029102a200c5de3f4b2b1a0b5e6c7d8e9f0a1b2c3d4e5f",
          "payload_root_sha256": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
        }
      ]
    }
  ],
  "schema": {
    "manifest_id": "libro",
    "schema_sha256": "4e5f6a7b8c9d0e1f2a3b4c5d6e7f8091a2b3c4d5e6f708192a3b4c5d6e7f8091",
    "package_version": "0.4.2",
    "recorded_at": "2026-10-20T10:15:05Z",
    "recorded_by_node": "node-7f3a9c",
    "settings": [
      {
        "key": "root",
        "label": "Library root",
        "type": "string",
        "values": [],
        "default": "/srv/audiobooks",
        "required": true,
        "description": "Directory the finished audiobooks are written to."
      }
    ]
  }
}
```

Schema:

```json
{
  "allOf": [
    {
      "type": "object",
      "description": "The plugin with its versions.",
      "required": [
        "plugin_key",
        "versions"
      ],
      "properties": {
        "plugin_key": {
          "type": "string",
          "description": "Canonical plugin key.",
          "example": "libro"
        },
        "id": {
          "type": [
            "string",
            "null"
          ],
          "description": "Manifest id (display alias) of the active version, or of the newest\none when none is active; `null` without versions.",
          "example": "libro"
        },
        "active_version": {
          "type": [
            "string",
            "null"
          ],
          "description": "Active version; `null` when the plugin is installed but not\nactivated.",
          "example": "0.4.2"
        },
        "revision": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Compare-and-swap revision of the active-version pointer; `null`\nwhen nothing is active.",
          "example": 3
        },
        "versions": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One installed version of a plugin, as the API renders it.",
            "required": [
              "plugin_key",
              "id",
              "version",
              "active",
              "approved",
              "publisher_verified",
              "coordinate",
              "installed_at",
              "installed_by",
              "artifacts"
            ],
            "properties": {
              "plugin_key": {
                "type": "string",
                "description": "Canonical plugin key.",
                "example": "libro"
              },
              "id": {
                "type": "string",
                "description": "Manifest id (display alias).",
                "example": "libro"
              },
              "version": {
                "type": "string",
                "description": "Semver package version.",
                "example": "0.4.2"
              },
              "active": {
                "type": "boolean",
                "description": "True when this is the plugin's active version.",
                "example": true
              },
              "approved": {
                "type": "boolean",
                "description": "True once an operator approved the version's consent request.",
                "example": true
              },
              "publisher_verified": {
                "type": "boolean",
                "description": "True when the publisher was verified at install time.",
                "example": true
              },
              "coordinate": {
                "type": "string",
                "description": "Package coordinate the version was resolved from; empty for a local\narchive.",
                "example": "registry:plugins.example#source-libro@0.4.2"
              },
              "installed_at": {
                "type": "string",
                "description": "RFC 3339 install time.",
                "example": "2026-10-20T10:15:00Z"
              },
              "installed_by": {
                "type": "string",
                "description": "Audit identity of the installer.",
                "example": "operator"
              },
              "host_payload_root_sha256": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Payload-root digest of the artifact this host would run; `null` when\nno artifact fits the host's target.",
                "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
              },
              "artifacts": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "One downloadable artifact of an installed version, as the API renders\nit.",
                  "required": [
                    "target",
                    "url",
                    "archive_sha256",
                    "payload_root_sha256"
                  ],
                  "properties": {
                    "target": {
                      "type": "string",
                      "description": "Bookclerk target (`linux-x64-gnu`, …; `any` for workerd packages).",
                      "example": "linux-x64-gnu"
                    },
                    "url": {
                      "type": "string",
                      "description": "Where nodes fetch the archive: `https:`, `file:`, or\n`blob:sha256:<hex>`.",
                      "example": "https://plugins.example/source-libro-0.4.2-linux-x64-gnu.tar.gz"
                    },
                    "archive_sha256": {
                      "type": "string",
                      "description": "Lowercase hex SHA-256 of the archive bytes.",
                      "example": "9b74c9897bac770ffc029102a200c5de3f4b2b1a0b5e6c7d8e9f0a1b2c3d4e5f"
                    },
                    "payload_root_sha256": {
                      "type": "string",
                      "description": "Payload-root digest of the extracted plugin tree.",
                      "example": "1f2e3d4c5b6a79880970a6b5c4d3e2f1a0b9c8d7e6f5a4b3c2d1e0f9a8b7c6d5"
                    }
                  }
                },
                "description": "Per-target artifacts."
              }
            }
          },
          "description": "Every installed version, newest first."
        }
      }
    },
    {
      "type": "object",
      "properties": {
        "schema": {
          "oneOf": [
            {
              "type": "object",
              "description": "The pinned settings schema; `null` until a node pins one.",
              "required": [
                "manifest_id",
                "schema_sha256",
                "package_version",
                "recorded_at",
                "recorded_by_node",
                "settings"
              ],
              "properties": {
                "manifest_id": {
                  "type": "string",
                  "description": "Manifest id (display alias) at pin time.",
                  "example": "libro"
                },
                "schema_sha256": {
                  "type": "string",
                  "description": "Fingerprint of the normalized schema.",
                  "example": "4e5f6a7b8c9d0e1f2a3b4c5d6e7f8091a2b3c4d5e6f708192a3b4c5d6e7f8091"
                },
                "package_version": {
                  "type": "string",
                  "description": "Semver package version that produced the pinned schema.",
                  "example": "0.4.2"
                },
                "recorded_at": {
                  "type": "string",
                  "description": "RFC 3339 time of the last pin.",
                  "example": "2026-10-20T10:15:05Z"
                },
                "recorded_by_node": {
                  "type": "string",
                  "description": "Node that recorded the last pin.",
                  "example": "node-7f3a9c"
                },
                "settings": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One setting a plugin declares in its `plugin.toml` `[[settings]]`:\nits key, value type, default and help text.",
                    "required": [
                      "key",
                      "type"
                    ],
                    "properties": {
                      "key": {
                        "type": "string",
                        "description": "Setting key (`[a-z][a-z0-9_]*`, at most 64 bytes).",
                        "example": "root"
                      },
                      "label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Operator-facing label; omitted when the key is shown.",
                        "example": "Library root"
                      },
                      "type": {
                        "type": "string",
                        "description": "Value type: `string`, `bool`, `integer`, `enum` or `secret`.",
                        "enum": [
                          "string",
                          "bool",
                          "integer",
                          "enum",
                          "secret"
                        ],
                        "example": "string"
                      },
                      "values": {
                        "type": [
                          "array",
                          "null"
                        ],
                        "items": {
                          "type": "object",
                          "description": "One allowed value of an `enum` setting: the stored id and its label.",
                          "required": [
                            "id"
                          ],
                          "properties": {
                            "id": {
                              "type": "string",
                              "description": "Stored value.",
                              "example": "sync_missing"
                            },
                            "label": {
                              "type": [
                                "string",
                                "null"
                              ],
                              "description": "Operator-facing label; omitted when the id is shown.",
                              "example": "Sync missing"
                            }
                          }
                        },
                        "description": "Allowed values of an `enum` setting; omitted for other kinds."
                      },
                      "default": {
                        "description": "Default applied when the instance document has no value; omitted when there is none. A string, boolean or integer, as the setting's `type` says.",
                        "examples": [
                          "/srv/audiobooks"
                        ]
                      },
                      "required": {
                        "type": [
                          "boolean",
                          "null"
                        ],
                        "description": "True when the instance cannot run without a value; omitted when\nfalse.",
                        "example": true
                      },
                      "description": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Help text shown beside the setting; omitted when there is none.",
                        "example": "Directory the finished audiobooks are written to."
                      }
                    }
                  },
                  "description": "Declared settings."
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
  ],
  "description": "`GET /api/config/plugins/{key}` body: the plugin and its pinned schema."
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

Status `404`: No such plugin.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `500`: The configuration could not be read.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/config/plugins/{key}`

`DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
cleared, placements dropped, every deployment `absent`).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Responses

- `200` — The plugin was uninstalled. Body: [`PluginUninstalled`](schemas.md#pluginuninstalled) (`application/json`)
  - `plugin_key` *string*, **required** — Canonical plugin key.
  - `uninstalled` *boolean*, **required** — True when an active-version pointer existed and was cleared.
  - `deployments_undeployed` *integer*, **required** — Deployment rows switched to `absent` across the plugin's instances. At least 0.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — A deployment changed under the request. The install was not cleared, but deployments of earlier instances may already be stopped; retry the request. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugins/<key>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugins/<key>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The plugin was uninstalled.

```json
{
  "plugin_key": "libro",
  "uninstalled": true,
  "deployments_undeployed": 2
}
```

Schema:

```json
{
  "type": "object",
  "description": "`DELETE /api/config/plugins/{key}` body.",
  "required": [
    "plugin_key",
    "uninstalled",
    "deployments_undeployed"
  ],
  "properties": {
    "plugin_key": {
      "type": "string",
      "description": "Canonical plugin key.",
      "example": "libro"
    },
    "uninstalled": {
      "type": "boolean",
      "description": "True when an active-version pointer existed and was cleared.",
      "example": true
    },
    "deployments_undeployed": {
      "type": "integer",
      "description": "Deployment rows switched to `absent` across the plugin's instances.",
      "example": 2,
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

Status `404`: No such plugin.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `409`: A deployment changed under the request. The install was not cleared, but deployments of earlier instances may already be stopped; retry the request.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `PUT /api/config/plugins/{key}/active`

`PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Request body

`application/json`: [`PutActiveVersion`](schemas.md#putactiveversion)

- `version` *string*, **required** — Installed version to activate.

### Responses

- `200` — The active version after the change. Body: [`ActiveVersionChanged`](schemas.md#activeversionchanged) (`application/json`)
  - `plugin_key` *string*, **required** — Canonical plugin key.
  - `version` *string*, **required** — Version now active.
  - `revision` *integer (int64)*, **required** — Compare-and-swap revision of the pointer after the move.
- `400` — The body is invalid. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin or version. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>/active' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"version": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>/active' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"version": "string"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugins/<key>/active",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"version": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>/active", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"version": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugins/<key>/active")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"version": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The active version after the change.

```json
{
  "plugin_key": "libro",
  "version": "0.4.2",
  "revision": 4
}
```

Schema:

```json
{
  "type": "object",
  "description": "`PUT /api/config/plugins/{key}/active` body: the pointer after the move.",
  "required": [
    "plugin_key",
    "version",
    "revision"
  ],
  "properties": {
    "plugin_key": {
      "type": "string",
      "description": "Canonical plugin key.",
      "example": "libro"
    },
    "version": {
      "type": "string",
      "description": "Version now active.",
      "example": "0.4.2"
    },
    "revision": {
      "type": "integer",
      "format": "int64",
      "description": "Compare-and-swap revision of the pointer after the move.",
      "example": 4
    }
  }
}
```

Status `400`: The body is invalid.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such plugin or version.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/config/plugins/{key}/versions/{version}/approve`

`POST /api/config/plugins/{key}/versions/{version}/approve`: record the
version's requested consent as approved for every node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.
- `version` (path) *string*, **required** — Semver version.

### Responses

- `200` — The version's grant was approved. Body: [`PluginVersionApproved`](schemas.md#pluginversionapproved) (`application/json`)
  - `plugin_key` *string*, **required** — Canonical plugin key.
  - `version` *string*, **required** — Semver version whose grant was approved.
  - `approved` *boolean*, **required** — Always true: a version that cannot be approved is an error instead.
  - `grant` *object*, **required** — The consent grant recorded for every node: the entrypoints, events, jobs, network domains and TCP targets the manifest asked for.
- `400` — The version's manifest does not parse. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin or version. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The configuration could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The version's grant was approved.

```json
{
  "plugin_key": "libro",
  "version": "0.4.2",
  "approved": true,
  "grant": {}
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/config/plugins/{key}/versions/{version}/approve` body.",
  "required": [
    "plugin_key",
    "version",
    "approved",
    "grant"
  ],
  "properties": {
    "plugin_key": {
      "type": "string",
      "description": "Canonical plugin key.",
      "example": "libro"
    },
    "version": {
      "type": "string",
      "description": "Semver version whose grant was approved.",
      "example": "0.4.2"
    },
    "approved": {
      "type": "boolean",
      "description": "Always true: a version that cannot be approved is an error instead.",
      "example": true
    },
    "grant": {
      "type": "object",
      "description": "The consent grant recorded for every node: the entrypoints, events,\njobs, network domains and TCP targets the manifest asked for."
    }
  }
}
```

Status `400`: The version's manifest does not parse.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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

Status `404`: No such plugin or version.

```json
{
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
  "error": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
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
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/plugins/{id}/consent`

Loads the consent dialog for one plugin, including coverage against the stored grant.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin id.
- `plugin_key` (query) *string* — Canonical PluginKey; when set, wins over the path alias.

### Responses

- `200` — What the plugin asks for and what is granted. Body: [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`)
  - `plugin_id` *string*, **required** — Plugin id whose grant is being reviewed.
  - `plugin_key` *string*, **required** — Canonical PluginKey for this install (empty when unknown).
  - `provenance` *string*, **required** — Host-evaluated provenance label.
  - `runtime` *string*, **required** — Guest runtime (`native` or `workerd`) — drives which controls are enforceable.
  - `request` *[`PluginGrantView`](schemas.md#plugingrantview)*, **required** — Manifest-derived grant the plugin is asking for.
  - `covered` *boolean*, **required** — True when an existing on-disk grant already satisfies the request.
  - `summary` *array of string*, **required** — Operator-facing bullet list of requested capabilities.
  - `existing` *[`PluginGrantView`](schemas.md#plugingrantview) or null* — Previously approved grant, when one is stored.
  - `brand` *[`PluginConsentBrand`](schemas.md#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
  - `limits` *[`PluginConsentLimits`](schemas.md#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The grant could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<id>/consent' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<id>/consent' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/plugins/<id>/consent",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<id>/consent", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/plugins/<id>/consent")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the plugin asks for and what is granted.

```json
{
  "plugin_id": "string",
  "plugin_key": "string",
  "provenance": "string",
  "runtime": "string",
  "request": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "covered": true,
  "summary": [
    "string"
  ],
  "existing": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "brand": {
    "name": "string",
    "bg": "string",
    "fg": "string",
    "accent": "string",
    "logo": "string"
  },
  "limits": {
    "cpu_ms": 0,
    "subrequests": 0,
    "max_cpu_ms": 0,
    "max_subrequests": 0,
    "disk_mib": 0,
    "max_disk_mib": 0,
    "memory_mib": 0,
    "max_memory_mib": 0,
    "cpu_cores": 0,
    "max_cpu_cores": 0,
    "jail_cpu_cores": 0,
    "extra_processes": 0,
    "max_extra_processes": 0,
    "known_bindings": [
      "string"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Consent dialog payload for one plugin.",
  "required": [
    "plugin_id",
    "plugin_key",
    "provenance",
    "runtime",
    "request",
    "covered",
    "summary",
    "limits"
  ],
  "properties": {
    "plugin_id": {
      "type": "string",
      "description": "Plugin id whose grant is being reviewed."
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical PluginKey for this install (empty when unknown)."
    },
    "provenance": {
      "type": "string",
      "description": "Host-evaluated provenance label."
    },
    "runtime": {
      "type": "string",
      "description": "Guest runtime (`native` or `workerd`) — drives which controls are enforceable."
    },
    "request": {
      "type": "object",
      "description": "Manifest-derived grant the plugin is asking for.",
      "required": [
        "pluginKey",
        "pluginId",
        "entrypoints",
        "producers",
        "consumers",
        "jobs",
        "networkMode",
        "domains",
        "manifestDomains",
        "operatorAddedDomains",
        "operatorDeniedDomains",
        "tcp",
        "operatorAddedTcp",
        "operatorDeniedTcp",
        "addressCidrs",
        "operatorAddedCidrs",
        "operatorDeniedCidrs",
        "allowUndeclaredPublicRedirects",
        "grantRevision",
        "authorityRevision",
        "bindings",
        "compatibilityFlags",
        "approvedAt"
      ],
      "properties": {
        "pluginKey": {
          "type": "string",
          "description": "Canonical provenance-qualified PluginKey."
        },
        "pluginId": {
          "type": "string",
          "description": "Plugin id this grant applies to."
        },
        "entrypoints": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
        },
        "producers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Event types the operator approved the plugin to publish."
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "object"
          },
          "description": "Approved event-consumer subscriptions."
        },
        "jobs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Approved job trigger types."
        },
        "networkMode": {
          "type": "string",
          "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
        },
        "domains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Hostnames the operator approved for egress."
        },
        "manifestDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Destinations requested by the current manifest."
        },
        "operatorAddedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added destinations beyond the manifest."
        },
        "operatorDeniedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied destinations still listed by the manifest."
        },
        "tcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Effective raw TCP grants."
        },
        "operatorAddedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-added TCP grants."
        },
        "operatorDeniedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-denied TCP grants still listed by the manifest."
        },
        "addressCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Effective address-space CIDRs."
        },
        "operatorAddedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added CIDRs."
        },
        "operatorDeniedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied CIDRs still listed by the manifest."
        },
        "allowUndeclaredPublicRedirects": {
          "type": "boolean",
          "description": "Undeclared public redirect permission."
        },
        "grantRevision": {
          "type": "string",
          "description": "SHA-256 of persisted operator consent (no host overlays)."
        },
        "authorityRevision": {
          "type": "string",
          "description": "SHA-256 of the canonical effective authority."
        },
        "bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Host bindings the operator approved."
        },
        "compatibilityFlags": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Extra compatibility flags stored on the grant."
        },
        "approvedAt": {
          "type": "string",
          "description": "RFC 3339 timestamp when the grant was last approved."
        },
        "cpuMs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd subrequest budget.",
          "minimum": 0
        },
        "diskMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "memoryMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional jail memory in mebibytes.",
          "minimum": 0
        },
        "cpuCores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Jail CPU in cores (2 d.p.); absent for workerd."
        },
        "extraProcesses": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Extra process/thread budget beyond launcher overhead; native only.",
          "minimum": 0
        }
      }
    },
    "covered": {
      "type": "boolean",
      "description": "True when an existing on-disk grant already satisfies the request."
    },
    "summary": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Operator-facing bullet list of requested capabilities."
    },
    "existing": {
      "oneOf": [
        {
          "type": "object",
          "description": "Previously approved grant, when one is stored.",
          "required": [
            "pluginKey",
            "pluginId",
            "entrypoints",
            "producers",
            "consumers",
            "jobs",
            "networkMode",
            "domains",
            "manifestDomains",
            "operatorAddedDomains",
            "operatorDeniedDomains",
            "tcp",
            "operatorAddedTcp",
            "operatorDeniedTcp",
            "addressCidrs",
            "operatorAddedCidrs",
            "operatorDeniedCidrs",
            "allowUndeclaredPublicRedirects",
            "grantRevision",
            "authorityRevision",
            "bindings",
            "compatibilityFlags",
            "approvedAt"
          ],
          "properties": {
            "pluginKey": {
              "type": "string",
              "description": "Canonical provenance-qualified PluginKey."
            },
            "pluginId": {
              "type": "string",
              "description": "Plugin id this grant applies to."
            },
            "entrypoints": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
            },
            "producers": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Event types the operator approved the plugin to publish."
            },
            "consumers": {
              "type": "array",
              "items": {
                "type": "object"
              },
              "description": "Approved event-consumer subscriptions."
            },
            "jobs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Approved job trigger types."
            },
            "networkMode": {
              "type": "string",
              "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
            },
            "domains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Hostnames the operator approved for egress."
            },
            "manifestDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Destinations requested by the current manifest."
            },
            "operatorAddedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added destinations beyond the manifest."
            },
            "operatorDeniedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied destinations still listed by the manifest."
            },
            "tcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Effective raw TCP grants."
            },
            "operatorAddedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-added TCP grants."
            },
            "operatorDeniedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-denied TCP grants still listed by the manifest."
            },
            "addressCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Effective address-space CIDRs."
            },
            "operatorAddedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added CIDRs."
            },
            "operatorDeniedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied CIDRs still listed by the manifest."
            },
            "allowUndeclaredPublicRedirects": {
              "type": "boolean",
              "description": "Undeclared public redirect permission."
            },
            "grantRevision": {
              "type": "string",
              "description": "SHA-256 of persisted operator consent (no host overlays)."
            },
            "authorityRevision": {
              "type": "string",
              "description": "SHA-256 of the canonical effective authority."
            },
            "bindings": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Host bindings the operator approved."
            },
            "compatibilityFlags": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Extra compatibility flags stored on the grant."
            },
            "approvedAt": {
              "type": "string",
              "description": "RFC 3339 timestamp when the grant was last approved."
            },
            "cpuMs": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd CPU budget in milliseconds.",
              "minimum": 0
            },
            "subrequests": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd subrequest budget.",
              "minimum": 0
            },
            "diskMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional plugin state disk budget in mebibytes.",
              "minimum": 0
            },
            "memoryMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional jail memory in mebibytes.",
              "minimum": 0
            },
            "cpuCores": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Jail CPU in cores (2 d.p.); absent for workerd."
            },
            "extraProcesses": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Extra process/thread budget beyond launcher overhead; native only.",
              "minimum": 0
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "brand": {
      "oneOf": [
        {
          "type": "object",
          "description": "Optional portal or manifest branding for the dialog.",
          "required": [
            "name"
          ],
          "properties": {
            "name": {
              "type": "string",
              "description": "Display name from the portal brand or plugin manifest."
            },
            "bg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Background CSS color from the portal brand, when known."
            },
            "fg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Foreground CSS color from the portal brand, when known."
            },
            "accent": {
              "type": [
                "string",
                "null"
              ],
              "description": "Accent CSS color from the portal brand, when known."
            },
            "logo": {
              "type": [
                "string",
                "null"
              ],
              "description": "Logo URL or host-served embed path."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "limits": {
      "type": "object",
      "description": "Host-capped resource defaults (workerd budgets + shared disk). Always set.",
      "required": [
        "cpu_ms",
        "subrequests",
        "max_cpu_ms",
        "max_subrequests",
        "disk_mib",
        "max_disk_mib",
        "memory_mib",
        "max_memory_mib",
        "cpu_cores",
        "max_cpu_cores",
        "extra_processes",
        "max_extra_processes",
        "known_bindings"
      ],
      "properties": {
        "cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd subrequest budget.",
          "minimum": 0
        },
        "max_cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd CPU milliseconds.",
          "minimum": 0
        },
        "max_subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd subrequests.",
          "minimum": 0
        },
        "disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "max_disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for plugin state disk in mebibytes.",
          "minimum": 0
        },
        "memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default jail memory in mebibytes.",
          "minimum": 0
        },
        "max_memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for jail memory in mebibytes.",
          "minimum": 0
        },
        "cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Manifest / default jail CPU in cores (2 d.p.)."
        },
        "max_cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Host max jail CPU in cores (2 d.p.)."
        },
        "jail_cpu_cores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Optional `[plugins.jail]` ceiling in cores."
        },
        "extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Default extra process/thread budget (native consent).",
          "minimum": 0
        },
        "max_extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for extra process budget.",
          "minimum": 0
        },
        "known_bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Suggested / default host bindings operators may grant."
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

Status `404`: No such plugin.

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

Status `500`: The grant could not be read.

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

## `POST /api/plugins/{id}/consent`

Persists an approved plugin grant after validating operator overrides.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin id.
- `plugin_key` (query) *string* — Canonical PluginKey; when set, wins over the path alias.

### Request body

`application/json`: [`PluginConsentApproveRequest`](schemas.md#pluginconsentapproverequest)

- `approve` *boolean*, **required** — Must be true; a false value is rejected with 400.
- `grant` *[`PluginGrantOverride`](schemas.md#plugingrantoverride) or null* — Optional operator overrides applied on top of the manifest request.

### Responses

- `200` — The grant was stored. Body: [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`)
  - `plugin_id` *string*, **required** — Plugin id whose grant is being reviewed.
  - `plugin_key` *string*, **required** — Canonical PluginKey for this install (empty when unknown).
  - `provenance` *string*, **required** — Host-evaluated provenance label.
  - `runtime` *string*, **required** — Guest runtime (`native` or `workerd`) — drives which controls are enforceable.
  - `request` *[`PluginGrantView`](schemas.md#plugingrantview)*, **required** — Manifest-derived grant the plugin is asking for.
  - `covered` *boolean*, **required** — True when an existing on-disk grant already satisfies the request.
  - `summary` *array of string*, **required** — Operator-facing bullet list of requested capabilities.
  - `existing` *[`PluginGrantView`](schemas.md#plugingrantview) or null* — Previously approved grant, when one is stored.
  - `brand` *[`PluginConsentBrand`](schemas.md#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
  - `limits` *[`PluginConsentLimits`](schemas.md#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.
- `400` — An override is not allowed. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The grant could not be stored. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<id>/consent' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"approve": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<id>/consent' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"approve": true}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/plugins/<id>/consent",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"approve": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<id>/consent", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"approve": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/plugins/<id>/consent")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"approve": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The grant was stored.

```json
{
  "plugin_id": "string",
  "plugin_key": "string",
  "provenance": "string",
  "runtime": "string",
  "request": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "covered": true,
  "summary": [
    "string"
  ],
  "existing": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "brand": {
    "name": "string",
    "bg": "string",
    "fg": "string",
    "accent": "string",
    "logo": "string"
  },
  "limits": {
    "cpu_ms": 0,
    "subrequests": 0,
    "max_cpu_ms": 0,
    "max_subrequests": 0,
    "disk_mib": 0,
    "max_disk_mib": 0,
    "memory_mib": 0,
    "max_memory_mib": 0,
    "cpu_cores": 0,
    "max_cpu_cores": 0,
    "jail_cpu_cores": 0,
    "extra_processes": 0,
    "max_extra_processes": 0,
    "known_bindings": [
      "string"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Consent dialog payload for one plugin.",
  "required": [
    "plugin_id",
    "plugin_key",
    "provenance",
    "runtime",
    "request",
    "covered",
    "summary",
    "limits"
  ],
  "properties": {
    "plugin_id": {
      "type": "string",
      "description": "Plugin id whose grant is being reviewed."
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical PluginKey for this install (empty when unknown)."
    },
    "provenance": {
      "type": "string",
      "description": "Host-evaluated provenance label."
    },
    "runtime": {
      "type": "string",
      "description": "Guest runtime (`native` or `workerd`) — drives which controls are enforceable."
    },
    "request": {
      "type": "object",
      "description": "Manifest-derived grant the plugin is asking for.",
      "required": [
        "pluginKey",
        "pluginId",
        "entrypoints",
        "producers",
        "consumers",
        "jobs",
        "networkMode",
        "domains",
        "manifestDomains",
        "operatorAddedDomains",
        "operatorDeniedDomains",
        "tcp",
        "operatorAddedTcp",
        "operatorDeniedTcp",
        "addressCidrs",
        "operatorAddedCidrs",
        "operatorDeniedCidrs",
        "allowUndeclaredPublicRedirects",
        "grantRevision",
        "authorityRevision",
        "bindings",
        "compatibilityFlags",
        "approvedAt"
      ],
      "properties": {
        "pluginKey": {
          "type": "string",
          "description": "Canonical provenance-qualified PluginKey."
        },
        "pluginId": {
          "type": "string",
          "description": "Plugin id this grant applies to."
        },
        "entrypoints": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
        },
        "producers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Event types the operator approved the plugin to publish."
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "object"
          },
          "description": "Approved event-consumer subscriptions."
        },
        "jobs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Approved job trigger types."
        },
        "networkMode": {
          "type": "string",
          "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
        },
        "domains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Hostnames the operator approved for egress."
        },
        "manifestDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Destinations requested by the current manifest."
        },
        "operatorAddedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added destinations beyond the manifest."
        },
        "operatorDeniedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied destinations still listed by the manifest."
        },
        "tcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Effective raw TCP grants."
        },
        "operatorAddedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-added TCP grants."
        },
        "operatorDeniedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-denied TCP grants still listed by the manifest."
        },
        "addressCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Effective address-space CIDRs."
        },
        "operatorAddedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added CIDRs."
        },
        "operatorDeniedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied CIDRs still listed by the manifest."
        },
        "allowUndeclaredPublicRedirects": {
          "type": "boolean",
          "description": "Undeclared public redirect permission."
        },
        "grantRevision": {
          "type": "string",
          "description": "SHA-256 of persisted operator consent (no host overlays)."
        },
        "authorityRevision": {
          "type": "string",
          "description": "SHA-256 of the canonical effective authority."
        },
        "bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Host bindings the operator approved."
        },
        "compatibilityFlags": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Extra compatibility flags stored on the grant."
        },
        "approvedAt": {
          "type": "string",
          "description": "RFC 3339 timestamp when the grant was last approved."
        },
        "cpuMs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd subrequest budget.",
          "minimum": 0
        },
        "diskMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "memoryMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional jail memory in mebibytes.",
          "minimum": 0
        },
        "cpuCores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Jail CPU in cores (2 d.p.); absent for workerd."
        },
        "extraProcesses": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Extra process/thread budget beyond launcher overhead; native only.",
          "minimum": 0
        }
      }
    },
    "covered": {
      "type": "boolean",
      "description": "True when an existing on-disk grant already satisfies the request."
    },
    "summary": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Operator-facing bullet list of requested capabilities."
    },
    "existing": {
      "oneOf": [
        {
          "type": "object",
          "description": "Previously approved grant, when one is stored.",
          "required": [
            "pluginKey",
            "pluginId",
            "entrypoints",
            "producers",
            "consumers",
            "jobs",
            "networkMode",
            "domains",
            "manifestDomains",
            "operatorAddedDomains",
            "operatorDeniedDomains",
            "tcp",
            "operatorAddedTcp",
            "operatorDeniedTcp",
            "addressCidrs",
            "operatorAddedCidrs",
            "operatorDeniedCidrs",
            "allowUndeclaredPublicRedirects",
            "grantRevision",
            "authorityRevision",
            "bindings",
            "compatibilityFlags",
            "approvedAt"
          ],
          "properties": {
            "pluginKey": {
              "type": "string",
              "description": "Canonical provenance-qualified PluginKey."
            },
            "pluginId": {
              "type": "string",
              "description": "Plugin id this grant applies to."
            },
            "entrypoints": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
            },
            "producers": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Event types the operator approved the plugin to publish."
            },
            "consumers": {
              "type": "array",
              "items": {
                "type": "object"
              },
              "description": "Approved event-consumer subscriptions."
            },
            "jobs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Approved job trigger types."
            },
            "networkMode": {
              "type": "string",
              "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
            },
            "domains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Hostnames the operator approved for egress."
            },
            "manifestDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Destinations requested by the current manifest."
            },
            "operatorAddedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added destinations beyond the manifest."
            },
            "operatorDeniedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied destinations still listed by the manifest."
            },
            "tcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Effective raw TCP grants."
            },
            "operatorAddedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-added TCP grants."
            },
            "operatorDeniedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-denied TCP grants still listed by the manifest."
            },
            "addressCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Effective address-space CIDRs."
            },
            "operatorAddedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added CIDRs."
            },
            "operatorDeniedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied CIDRs still listed by the manifest."
            },
            "allowUndeclaredPublicRedirects": {
              "type": "boolean",
              "description": "Undeclared public redirect permission."
            },
            "grantRevision": {
              "type": "string",
              "description": "SHA-256 of persisted operator consent (no host overlays)."
            },
            "authorityRevision": {
              "type": "string",
              "description": "SHA-256 of the canonical effective authority."
            },
            "bindings": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Host bindings the operator approved."
            },
            "compatibilityFlags": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Extra compatibility flags stored on the grant."
            },
            "approvedAt": {
              "type": "string",
              "description": "RFC 3339 timestamp when the grant was last approved."
            },
            "cpuMs": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd CPU budget in milliseconds.",
              "minimum": 0
            },
            "subrequests": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd subrequest budget.",
              "minimum": 0
            },
            "diskMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional plugin state disk budget in mebibytes.",
              "minimum": 0
            },
            "memoryMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional jail memory in mebibytes.",
              "minimum": 0
            },
            "cpuCores": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Jail CPU in cores (2 d.p.); absent for workerd."
            },
            "extraProcesses": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Extra process/thread budget beyond launcher overhead; native only.",
              "minimum": 0
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "brand": {
      "oneOf": [
        {
          "type": "object",
          "description": "Optional portal or manifest branding for the dialog.",
          "required": [
            "name"
          ],
          "properties": {
            "name": {
              "type": "string",
              "description": "Display name from the portal brand or plugin manifest."
            },
            "bg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Background CSS color from the portal brand, when known."
            },
            "fg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Foreground CSS color from the portal brand, when known."
            },
            "accent": {
              "type": [
                "string",
                "null"
              ],
              "description": "Accent CSS color from the portal brand, when known."
            },
            "logo": {
              "type": [
                "string",
                "null"
              ],
              "description": "Logo URL or host-served embed path."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "limits": {
      "type": "object",
      "description": "Host-capped resource defaults (workerd budgets + shared disk). Always set.",
      "required": [
        "cpu_ms",
        "subrequests",
        "max_cpu_ms",
        "max_subrequests",
        "disk_mib",
        "max_disk_mib",
        "memory_mib",
        "max_memory_mib",
        "cpu_cores",
        "max_cpu_cores",
        "extra_processes",
        "max_extra_processes",
        "known_bindings"
      ],
      "properties": {
        "cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd subrequest budget.",
          "minimum": 0
        },
        "max_cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd CPU milliseconds.",
          "minimum": 0
        },
        "max_subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd subrequests.",
          "minimum": 0
        },
        "disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "max_disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for plugin state disk in mebibytes.",
          "minimum": 0
        },
        "memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default jail memory in mebibytes.",
          "minimum": 0
        },
        "max_memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for jail memory in mebibytes.",
          "minimum": 0
        },
        "cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Manifest / default jail CPU in cores (2 d.p.)."
        },
        "max_cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Host max jail CPU in cores (2 d.p.)."
        },
        "jail_cpu_cores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Optional `[plugins.jail]` ceiling in cores."
        },
        "extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Default extra process/thread budget (native consent).",
          "minimum": 0
        },
        "max_extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for extra process budget.",
          "minimum": 0
        },
        "known_bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Suggested / default host bindings operators may grant."
        }
      }
    }
  }
}
```

Status `400`: An override is not allowed.

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

Status `404`: No such plugin.

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

Status `500`: The grant could not be stored.

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

## `GET /api/plugins/{kind}/{id}/logo`

Serve an embedded `plugin.toml` `logo` file from under the plugin install root.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `kind` (path) *string*, **required** — Plugin kind (`source`, `destination`, `integration`, …).
- `id` (path) *string*, **required** — Plugin id.

### Responses

- `200` — The logo (PNG, or SVG served with a restrictive CSP). Body: [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/svg+xml`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The plugin has no logo. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The logo (PNG, or SVG served with a restrictive CSP). The body is binary (`image/png`).

Schema:

A `image/png` body: binary bytes.

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

Status `404`: The plugin has no logo.

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
