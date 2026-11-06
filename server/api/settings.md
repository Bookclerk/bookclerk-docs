# Settings

Daemon settings and user preferences.

Back to the [web API reference](README.md).

## `GET /api/settings`

Returns file-backed and runtime-effective Settings plus per-plugin groups.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The settings. Body: [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`)
  - `settings` *map of string to string*, **required** — Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document.
  - `effective` *map of string to string*, **required** — Runtime-effective values after the last successful reload (auth, plugins).
  - `plugins` *array of [`PluginSettingsGroup`](schemas.md#pluginsettingsgroup)*, **required** — Per-plugin setting groups rendered by the Settings UI.
  - `host_cpu_cores_max` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.; equals logical CPU count).
  - `jail_cpu_cores` *number (double) or null* — Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).
  - `output_revision` *integer (int64) or null* — Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The settings could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/settings' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/settings",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/settings", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/settings")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings.

```json
{
  "settings": {
    "daemon.auth.enabled": "true",
    "daemon.listen": "127.0.0.1:8787,[::1]:8787",
    "library.auto_acquire": "false"
  },
  "effective": {
    "daemon.auth.enabled": "true",
    "daemon.listen": "127.0.0.1:8787,[::1]:8787",
    "library.auto_acquire": "false"
  },
  "plugins": [
    {
      "id": "libro",
      "family": "source",
      "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro",
      "provenance": "platform_bundled",
      "logo": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ",
      "settings": [
        {
          "key": "container",
          "label": "Container",
          "value": "m4b",
          "value_type": "string"
        }
      ],
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "display_name": "Libro.fm (household)",
      "is_default": true,
      "config_revision": 4,
      "deployment": {
        "deployment_id": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
        "node_id": "node-7f3a9c",
        "desired": "present",
        "revision": 3,
        "status": "healthy",
        "detail": "",
        "applied_config_revision": 5,
        "observed_at": "2026-10-28T19:07:00Z"
      },
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
      },
      "present_on": [
        "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
      ]
    }
  ],
  "host_cpu_cores_max": 8.0,
  "jail_cpu_cores": 2.0,
  "output_revision": 7
}
```

Schema:

```json
{
  "type": "object",
  "description": "Settings page payload: file values, effective runtime, and plugin groups.",
  "required": [
    "settings",
    "effective",
    "plugins",
    "host_cpu_cores_max"
  ],
  "properties": {
    "settings": {
      "type": "object",
      "description": "Values as written in config.toml (and env overlays), plus `output.*`\nfrom the `core.output` database document.",
      "additionalProperties": {
        "type": "string"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "daemon.auth.enabled": "true",
        "daemon.listen": "127.0.0.1:8787,[::1]:8787",
        "library.auto_acquire": "false"
      }
    },
    "effective": {
      "type": "object",
      "description": "Runtime-effective values after the last successful reload (auth, plugins).",
      "additionalProperties": {
        "type": "string"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "daemon.auth.enabled": "true",
        "daemon.listen": "127.0.0.1:8787,[::1]:8787",
        "library.auto_acquire": "false"
      }
    },
    "plugins": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Settings UI group for one discovered or loaded plugin.",
        "required": [
          "id",
          "family",
          "settings"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Plugin id (`audible`, `sqlite`, and similar).",
            "example": "libro"
          },
          "family": {
            "type": "string",
            "description": "Handler family wire label (`source`, `integration`, `output`, `database`).",
            "example": "source"
          },
          "plugin_key": {
            "type": [
              "string",
              "null"
            ],
            "description": "Provenance-qualified PluginKey when this row came from a discovered install.",
            "example": "platform:bookclerk/bookclerk-plugin-source-libro#libro"
          },
          "provenance": {
            "type": [
              "string",
              "null"
            ],
            "description": "Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …).",
            "example": "platform_bundled"
          },
          "logo": {
            "type": [
              "string",
              "null"
            ],
            "description": "Same-origin logo for Settings list rows: the plugin's embedded logo,\nor a 128 px copy of the remote logo its manifest names.",
            "example": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
          },
          "settings": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One editable setting shown in the Settings UI.",
              "required": [
                "key",
                "label",
                "value",
                "value_type"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Dotted `config.toml` key for this option.",
                  "example": "container"
                },
                "label": {
                  "type": "string",
                  "description": "Operator-facing field caption.",
                  "example": "Container"
                },
                "value": {
                  "type": "string",
                  "description": "Current string form of the setting.",
                  "example": "m4b"
                },
                "value_type": {
                  "type": "string",
                  "description": "UI widget hint (`boolean`, `string`, or `number`).",
                  "example": "string"
                },
                "choices": {
                  "type": [
                    "array",
                    "null"
                  ],
                  "items": {
                    "type": "object",
                    "description": "One selectable value for a Settings dropdown.",
                    "required": [
                      "value",
                      "label"
                    ],
                    "properties": {
                      "value": {
                        "type": "string",
                        "description": "Stored config value; empty string means use the plugin default.",
                        "example": "m4b"
                      },
                      "label": {
                        "type": "string",
                        "description": "Operator-facing choice caption.",
                        "example": "M4B"
                      }
                    }
                  },
                  "description": "Optional dropdown values; omitted for free-form fields."
                },
                "secret": {
                  "type": "boolean",
                  "description": "True for a `secret` setting: `value` is never the stored value.",
                  "example": false
                },
                "secret_set": {
                  "type": [
                    "boolean",
                    "null"
                  ],
                  "description": "For a secret setting, whether the instance document names a sealed value.",
                  "example": true
                }
              }
            },
            "description": "Editable options for this plugin, including `.enabled`."
          },
          "plugin_instance_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Plugin instance this group edits; absent for a bootstrap database adapter.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "display_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing instance name, when the operator set one.",
            "example": "Libro.fm (household)"
          },
          "is_default": {
            "type": "boolean",
            "description": "True when a bare plugin key or alias routes to this instance.",
            "example": true
          },
          "config_revision": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Instance document revision for compare-and-swap writes.",
            "example": 4
          },
          "deployment": {
            "oneOf": [
              {
                "type": "object",
                "description": "This node's deployment of the instance, when one exists.",
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
              {
                "type": "null"
              }
            ]
          },
          "placement": {
            "oneOf": [
              {
                "type": "object",
                "description": "Scheduler placement; absent when the instance is hand-managed (the\nEnabled switch applies only then).",
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
          },
          "present_on": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Nodes whose deployment of the instance is `present`.",
            "example": [
              "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
            ]
          }
        }
      },
      "description": "Per-plugin setting groups rendered by the Settings UI."
    },
    "host_cpu_cores_max": {
      "type": "number",
      "format": "double",
      "description": "Host max jail CPU in cores (2 d.p.; equals logical CPU count).",
      "example": 8.0
    },
    "jail_cpu_cores": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).",
      "example": 2.0
    },
    "output_revision": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Revision of the `core.output` document the `output.*` values in\n`settings` were read from; send it back as `expected_output_revision`.\n`effective[\"output.revision\"]` is the revision this host has applied,\nwhich lags the document until the host's next wakeup.",
      "example": 7
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

Status `500`: The settings could not be read.

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

## `PATCH /api/settings`

Validates, writes, and reloads allowlisted Settings keys under `reload_lock`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PatchSettingsRequest`](schemas.md#patchsettingsrequest)

- `settings` *array of [`SettingsUpdate`](schemas.md#settingsupdate)*, **required** — Key/value pairs to validate, persist, and reload.
- `expected_output_revision` *integer (int64) or null* — `core.output` document revision the client read (`output_revision` in `GET /api/settings`). When set and stale, an `output.*` update is rejected with 409.

### Responses

- `200` — The settings after the change. Body: [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`)
  - `settings` *map of string to string*, **required** — Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document.
  - `effective` *map of string to string*, **required** — Runtime-effective values after the last successful reload (auth, plugins).
  - `plugins` *array of [`PluginSettingsGroup`](schemas.md#pluginsettingsgroup)*, **required** — Per-plugin setting groups rendered by the Settings UI.
  - `host_cpu_cores_max` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.; equals logical CPU count).
  - `jail_cpu_cores` *number (double) or null* — Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).
  - `output_revision` *integer (int64) or null* — Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup.
- `400` — A key is unknown or a value is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/plain`)
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
- `409` — `output.*` keys were sent with a stale `expected_output_revision`, or another write won; the body's `current_revision` is the revision to retry against (`revision_conflict`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The settings could not be written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"settings": [{"key": "library.auto_acquire", "value": "false"}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/settings' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"settings": [{"key": "library.auto_acquire", "value": "false"}]}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/settings",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"settings": [{"key": "library.auto_acquire", "value": "false"}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/settings", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"settings": [{"key": "library.auto_acquire", "value": "false"}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/settings")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"settings": [{"key": "library.auto_acquire", "value": "false"}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change.

```json
{
  "settings": {
    "daemon.auth.enabled": "true",
    "daemon.listen": "127.0.0.1:8787,[::1]:8787",
    "library.auto_acquire": "false"
  },
  "effective": {
    "daemon.auth.enabled": "true",
    "daemon.listen": "127.0.0.1:8787,[::1]:8787",
    "library.auto_acquire": "false"
  },
  "plugins": [
    {
      "id": "libro",
      "family": "source",
      "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro",
      "provenance": "platform_bundled",
      "logo": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ",
      "settings": [
        {
          "key": "container",
          "label": "Container",
          "value": "m4b",
          "value_type": "string"
        }
      ],
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "display_name": "Libro.fm (household)",
      "is_default": true,
      "config_revision": 4,
      "deployment": {
        "deployment_id": "8c1d2e3f-4a5b-4c6d-8e7f-9a0b1c2d3e4f",
        "node_id": "node-7f3a9c",
        "desired": "present",
        "revision": 3,
        "status": "healthy",
        "detail": "",
        "applied_config_revision": 5,
        "observed_at": "2026-10-28T19:07:00Z"
      },
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
      },
      "present_on": [
        "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
      ]
    }
  ],
  "host_cpu_cores_max": 8.0,
  "jail_cpu_cores": 2.0,
  "output_revision": 7
}
```

Schema:

```json
{
  "type": "object",
  "description": "Settings page payload: file values, effective runtime, and plugin groups.",
  "required": [
    "settings",
    "effective",
    "plugins",
    "host_cpu_cores_max"
  ],
  "properties": {
    "settings": {
      "type": "object",
      "description": "Values as written in config.toml (and env overlays), plus `output.*`\nfrom the `core.output` database document.",
      "additionalProperties": {
        "type": "string"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "daemon.auth.enabled": "true",
        "daemon.listen": "127.0.0.1:8787,[::1]:8787",
        "library.auto_acquire": "false"
      }
    },
    "effective": {
      "type": "object",
      "description": "Runtime-effective values after the last successful reload (auth, plugins).",
      "additionalProperties": {
        "type": "string"
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "daemon.auth.enabled": "true",
        "daemon.listen": "127.0.0.1:8787,[::1]:8787",
        "library.auto_acquire": "false"
      }
    },
    "plugins": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Settings UI group for one discovered or loaded plugin.",
        "required": [
          "id",
          "family",
          "settings"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Plugin id (`audible`, `sqlite`, and similar).",
            "example": "libro"
          },
          "family": {
            "type": "string",
            "description": "Handler family wire label (`source`, `integration`, `output`, `database`).",
            "example": "source"
          },
          "plugin_key": {
            "type": [
              "string",
              "null"
            ],
            "description": "Provenance-qualified PluginKey when this row came from a discovered install.",
            "example": "platform:bookclerk/bookclerk-plugin-source-libro#libro"
          },
          "provenance": {
            "type": [
              "string",
              "null"
            ],
            "description": "Host-evaluated provenance (`platform_bundled`, `verified_staged`, `installed`, …).",
            "example": "platform_bundled"
          },
          "logo": {
            "type": [
              "string",
              "null"
            ],
            "description": "Same-origin logo for Settings list rows: the plugin's embedded logo,\nor a 128 px copy of the remote logo its manifest names.",
            "example": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
          },
          "settings": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One editable setting shown in the Settings UI.",
              "required": [
                "key",
                "label",
                "value",
                "value_type"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Dotted `config.toml` key for this option.",
                  "example": "container"
                },
                "label": {
                  "type": "string",
                  "description": "Operator-facing field caption.",
                  "example": "Container"
                },
                "value": {
                  "type": "string",
                  "description": "Current string form of the setting.",
                  "example": "m4b"
                },
                "value_type": {
                  "type": "string",
                  "description": "UI widget hint (`boolean`, `string`, or `number`).",
                  "example": "string"
                },
                "choices": {
                  "type": [
                    "array",
                    "null"
                  ],
                  "items": {
                    "type": "object",
                    "description": "One selectable value for a Settings dropdown.",
                    "required": [
                      "value",
                      "label"
                    ],
                    "properties": {
                      "value": {
                        "type": "string",
                        "description": "Stored config value; empty string means use the plugin default.",
                        "example": "m4b"
                      },
                      "label": {
                        "type": "string",
                        "description": "Operator-facing choice caption.",
                        "example": "M4B"
                      }
                    }
                  },
                  "description": "Optional dropdown values; omitted for free-form fields."
                },
                "secret": {
                  "type": "boolean",
                  "description": "True for a `secret` setting: `value` is never the stored value.",
                  "example": false
                },
                "secret_set": {
                  "type": [
                    "boolean",
                    "null"
                  ],
                  "description": "For a secret setting, whether the instance document names a sealed value.",
                  "example": true
                }
              }
            },
            "description": "Editable options for this plugin, including `.enabled`."
          },
          "plugin_instance_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Plugin instance this group edits; absent for a bootstrap database adapter.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "display_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing instance name, when the operator set one.",
            "example": "Libro.fm (household)"
          },
          "is_default": {
            "type": "boolean",
            "description": "True when a bare plugin key or alias routes to this instance.",
            "example": true
          },
          "config_revision": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Instance document revision for compare-and-swap writes.",
            "example": 4
          },
          "deployment": {
            "oneOf": [
              {
                "type": "object",
                "description": "This node's deployment of the instance, when one exists.",
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
              {
                "type": "null"
              }
            ]
          },
          "placement": {
            "oneOf": [
              {
                "type": "object",
                "description": "Scheduler placement; absent when the instance is hand-managed (the\nEnabled switch applies only then).",
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
          },
          "present_on": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Nodes whose deployment of the instance is `present`.",
            "example": [
              "6d1f2e3a-4b5c-4d6e-8f7a-9b0c1d2e3f4a"
            ]
          }
        }
      },
      "description": "Per-plugin setting groups rendered by the Settings UI."
    },
    "host_cpu_cores_max": {
      "type": "number",
      "format": "double",
      "description": "Host max jail CPU in cores (2 d.p.; equals logical CPU count).",
      "example": 8.0
    },
    "jail_cpu_cores": {
      "type": [
        "number",
        "null"
      ],
      "format": "double",
      "description": "Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).",
      "example": 2.0
    },
    "output_revision": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Revision of the `core.output` document the `output.*` values in\n`settings` were read from; send it back as `expected_output_revision`.\n`effective[\"output.revision\"]` is the revision this host has applied,\nwhich lags the document until the host's next wakeup.",
      "example": 7
    }
  }
}
```

Status `400`: A key is unknown or a value is invalid. As `text/plain`, when the body is not syntactically valid JSON.

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

Status `409`: `output.*` keys were sent with a stale `expected_output_revision`, or another write won; the body's `current_revision` is the revision to retry against (`revision_conflict`).

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `settings` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The settings could not be written.

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

## `POST /api/database/migrate`

Copies library rows between database plugins and optionally switches `[database].plugin`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`DatabaseMigrateRequest`](schemas.md#databasemigraterequest)

- `from` *string or null* — Source plugin id. Defaults to the active `[database].plugin` before reload.
- `to` *string*, **required** — Destination database plugin id to copy into.
- `dry_run` *boolean* — When true, count rows without writing.
- `force` *boolean* — When true, overwrite a destination that is not empty.
- `apply` *boolean* — Update `[database].plugin` in config.toml after a successful copy.

### Responses

- `200` — The rows were copied. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `400` — The target is invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`); string (`text/plain`)
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
- `409` — The migration target matches more than one database plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The copy failed. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/database/migrate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"to": "postgres"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/database/migrate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"to": "postgres"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/database/migrate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"to": "postgres"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/database/migrate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"to": "postgres"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/database/migrate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"to": "postgres"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The rows were copied.

```json
{
  "job_id": "",
  "message": "copied 1284 row(s) from `sqlite` to `postgres`; updated [database].plugin, wrote /home/bookclerk/BookclerkFiles/config.toml, and reloaded library connection",
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

Status `400`: The target is invalid. As `text/plain`, when the body is not syntactically valid JSON.

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

Status `409`: The migration target matches more than one database plugin.

```json
{
  "error": "conflict",
  "message": "Conflict",
  "status": 409
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `to` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The copy failed.

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

## `GET /api/preferences`

Loads preferences for the caller with a 3s timeout, creating defaults when missing.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's preferences. Body: [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`)
  - `default_view` *string*, **required** — Landing view id (`library`, `discover`, …) after normalization.
  - `disabled_shelves` *array of string*, **required** — Lowercased Discover shelf ids the user has hidden.
  - `discover_sort` *string*, **required** — Discover sort key (`relevance`, `popularity`, …).
  - `discover_sort_dir` *string*, **required** — Discover sort direction (`asc` or `desc`).
  - `discover_language` *string or null* — `null` = use browser language on the client.
  - `discover_excluded_sources` *array of string*, **required** — Lowercased store ids hidden from Discover results.
  - `theme` *string*, **required** — Appearance preference (`system`, `light`, or `dark`).
- `401` — No signed-in caller. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The preferences row did not load in time. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/preferences' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/preferences",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/preferences", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/preferences")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's preferences.

```json
{
  "default_view": "library",
  "disabled_shelves": [
    "chirp_deals",
    "from_store"
  ],
  "discover_sort": "relevance",
  "discover_sort_dir": "desc",
  "discover_language": "en",
  "discover_excluded_sources": [
    "chirp"
  ],
  "theme": "system"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Normalized Discover/library preferences for the authenticated subject.",
  "required": [
    "default_view",
    "disabled_shelves",
    "discover_sort",
    "discover_sort_dir",
    "discover_excluded_sources",
    "theme"
  ],
  "properties": {
    "default_view": {
      "type": "string",
      "description": "Landing view id (`library`, `discover`, …) after normalization.",
      "example": "library"
    },
    "disabled_shelves": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Lowercased Discover shelf ids the user has hidden.",
      "example": [
        "chirp_deals",
        "from_store"
      ]
    },
    "discover_sort": {
      "type": "string",
      "description": "Discover sort key (`relevance`, `popularity`, …).",
      "example": "relevance"
    },
    "discover_sort_dir": {
      "type": "string",
      "description": "Discover sort direction (`asc` or `desc`).",
      "example": "desc"
    },
    "discover_language": {
      "type": [
        "string",
        "null"
      ],
      "description": "`null` = use browser language on the client.",
      "example": "en"
    },
    "discover_excluded_sources": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Lowercased store ids hidden from Discover results.",
      "example": [
        "chirp"
      ]
    },
    "theme": {
      "type": "string",
      "description": "Appearance preference (`system`, `light`, or `dark`).",
      "example": "system"
    }
  }
}
```

Status `401`: No signed-in caller.

```text
Unauthorized
```

Schema:

A `text/plain` body: a string.

Status `500`: The request could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

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

Status `504`: The preferences row did not load in time.

```text
preferences lookup timed out
```

Schema:

A `text/plain` body: a string.

## `PATCH /api/preferences`

Merges a preferences PATCH onto the caller’s stored row and writes it back.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchPreferencesBody`](schemas.md#patchpreferencesbody)

- `default_view` *string or null* — Replacement landing view when set.
- `disabled_shelves` *array of string or null* — Replacement hidden-shelf list when set.
- `discover_sort` *string or null* — Replacement Discover sort key when set.
- `discover_sort_dir` *string or null* — Replacement Discover sort direction when set.
- `discover_language` *string or null* — Present + value sets language; present + `null` clears to browser default; omitted leaves unchanged.
- `discover_excluded_sources` *array of string or null* — Replacement excluded-store list when set.
- `theme` *string or null* — Replacement appearance preference when set (`system`, `light`, `dark`).

### Responses

- `200` — The preferences after the change. Body: [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`)
  - `default_view` *string*, **required** — Landing view id (`library`, `discover`, …) after normalization.
  - `disabled_shelves` *array of string*, **required** — Lowercased Discover shelf ids the user has hidden.
  - `discover_sort` *string*, **required** — Discover sort key (`relevance`, `popularity`, …).
  - `discover_sort_dir` *string*, **required** — Discover sort direction (`asc` or `desc`).
  - `discover_language` *string or null* — `null` = use browser language on the client.
  - `discover_excluded_sources` *array of string*, **required** — Lowercased store ids hidden from Discover results.
  - `theme` *string*, **required** — Appearance preference (`system`, `light`, or `dark`).
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — No signed-in caller. Body: string (`text/plain`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the patch (a wrong type or an unknown shape). Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The preferences row did not load or save in time. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/preferences' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/preferences",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/preferences", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/preferences")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The preferences after the change.

```json
{
  "default_view": "library",
  "disabled_shelves": [
    "chirp_deals",
    "from_store"
  ],
  "discover_sort": "relevance",
  "discover_sort_dir": "desc",
  "discover_language": "en",
  "discover_excluded_sources": [
    "chirp"
  ],
  "theme": "system"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Normalized Discover/library preferences for the authenticated subject.",
  "required": [
    "default_view",
    "disabled_shelves",
    "discover_sort",
    "discover_sort_dir",
    "discover_excluded_sources",
    "theme"
  ],
  "properties": {
    "default_view": {
      "type": "string",
      "description": "Landing view id (`library`, `discover`, …) after normalization.",
      "example": "library"
    },
    "disabled_shelves": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Lowercased Discover shelf ids the user has hidden.",
      "example": [
        "chirp_deals",
        "from_store"
      ]
    },
    "discover_sort": {
      "type": "string",
      "description": "Discover sort key (`relevance`, `popularity`, …).",
      "example": "relevance"
    },
    "discover_sort_dir": {
      "type": "string",
      "description": "Discover sort direction (`asc` or `desc`).",
      "example": "desc"
    },
    "discover_language": {
      "type": [
        "string",
        "null"
      ],
      "description": "`null` = use browser language on the client.",
      "example": "en"
    },
    "discover_excluded_sources": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Lowercased store ids hidden from Discover results.",
      "example": [
        "chirp"
      ]
    },
    "theme": {
      "type": "string",
      "description": "Appearance preference (`system`, `light`, or `dark`).",
      "example": "system"
    }
  }
}
```

Status `400`: The body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 10
```

Schema:

A `text/plain` body: a string.

Status `401`: No signed-in caller.

```text
Unauthorized
```

Schema:

A `text/plain` body: a string.

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the patch (a wrong type or an unknown shape).

```text
Failed to deserialize the JSON body into the target type: theme: invalid type: integer `3`, expected a string at line 1 column 11
```

Schema:

A `text/plain` body: a string.

Status `500`: The request could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

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

Status `504`: The preferences row did not load or save in time.

```text
preferences update timed out
```

Schema:

A `text/plain` body: a string.
