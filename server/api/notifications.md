# Notifications

Channels, routing rules, patron offers, the delivery log, and each user's own notification settings and inbox.

Back to the [web API reference](README.md).

## `GET /api/notifications/catalog`

Lists the notification types and the notifier instances the routing
screens pick from.

Types come from the host and from the `[[notifications.types]]` of the
plugins installed on this server; notifiers are the instances of
installed `notifier` plugins.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The types and notifiers. Body: [`NotificationCatalog`](schemas.md#notificationcatalog) (`application/json`)
  - `types` *array of [`NotificationTypeJson`](schemas.md#notificationtypejson)*, **required** — Host types first, then plugin types by id.
  - `notifiers` *array of [`NotifierJson`](schemas.md#notifierjson)*, **required** — Notifier instances installed on this server.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/catalog' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/catalog' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/notifications/catalog",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/catalog", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/notifications/catalog")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The types and notifiers.

```json
{
  "types": [
    {
      "id": "ntfy_relay:sync_failed",
      "audience": "operational",
      "severity": "warning",
      "title": "Sync failed",
      "source": "ntfy_relay"
    }
  ],
  "notifiers": [
    {
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "plugin_key": "plugin:telegram",
      "name": "Telegram",
      "channel": false,
      "personal": true,
      "address_fields": [
        {
          "key": "chat_id",
          "label": "Chat id",
          "required": true,
          "secret": false
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
  "description": "Everything the routing screens pick from.",
  "required": [
    "types",
    "notifiers"
  ],
  "properties": {
    "types": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One notification type a routing rule or a patron choice can name.",
        "required": [
          "id",
          "audience",
          "severity",
          "title",
          "source"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Qualified id: a host type (`test`) or `<plugin id>:<type>`.",
            "example": "ntfy_relay:sync_failed"
          },
          "audience": {
            "type": "string",
            "description": "Who it is for: `operational` (routed to channels) or `patron`\n(offered to patrons).",
            "example": "operational"
          },
          "severity": {
            "type": "string",
            "description": "Default severity: `info`, `warning` or `critical`.",
            "example": "warning"
          },
          "title": {
            "type": "string",
            "description": "Title shown in the routing matrix and on each notification.",
            "example": "Sync failed"
          },
          "source": {
            "type": "string",
            "description": "The plugin that raises it; empty for a host type.",
            "example": "ntfy_relay"
          }
        }
      },
      "description": "Host types first, then plugin types by id."
    },
    "notifiers": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One notifier instance.",
        "required": [
          "plugin_instance_id",
          "plugin_key",
          "name",
          "channel",
          "personal",
          "address_fields"
        ],
        "properties": {
          "plugin_instance_id": {
            "type": "string",
            "description": "Plugin instance id.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "plugin_key": {
            "type": "string",
            "description": "Canonical plugin key.",
            "example": "plugin:telegram"
          },
          "name": {
            "type": "string",
            "description": "Name shown in pickers.",
            "example": "Telegram"
          },
          "channel": {
            "type": "boolean",
            "description": "Whether administrators can make library channels on it.",
            "example": false
          },
          "personal": {
            "type": "boolean",
            "description": "Whether patrons can add their own address on it.",
            "example": true
          },
          "address_fields": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One field a patron fills in for a personal destination.",
              "required": [
                "key",
                "label",
                "required",
                "secret"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Key in the address document.",
                  "example": "chat_id"
                },
                "label": {
                  "type": "string",
                  "description": "Label shown on the form.",
                  "example": "Chat id"
                },
                "required": {
                  "type": "boolean",
                  "description": "Whether a destination must carry it.",
                  "example": true
                },
                "secret": {
                  "type": "boolean",
                  "description": "Whether the value is a secret: shown masked, never logged.",
                  "example": false
                }
              }
            },
            "description": "What a patron enters for a personal destination."
          }
        }
      },
      "description": "Notifier instances installed on this server."
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/notifications/channels`

Lists the library's channels.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The channels. Body: [`ChannelList`](schemas.md#channellist) (`application/json`)
  - `channels` *array of [`DestinationJson`](schemas.md#destinationjson)*, **required** — Oldest first.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/notifications/channels",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/notifications/channels")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The channels.

```json
{
  "channels": [
    {
      "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
      "kind": "channel",
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "label": "Ops room",
      "failure_count": 0,
      "paused_at": "2026-11-21T10:00:00+00:00",
      "paused_reason": "unauthenticated: the webhook was deleted",
      "created_at": "2026-11-20T09:00:00+00:00"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "The library's channels.",
  "required": [
    "channels"
  ],
  "properties": {
    "channels": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
        "required": [
          "id",
          "kind",
          "plugin_instance_id",
          "label",
          "failure_count",
          "created_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Destination id.",
            "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
          },
          "kind": {
            "type": "string",
            "description": "`channel` (the library's) or `personal` (a patron's).",
            "enum": [
              "channel",
              "personal"
            ]
          },
          "plugin_instance_id": {
            "type": "string",
            "description": "The notifier instance that carries it.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "label": {
            "type": "string",
            "description": "Name shown in pickers.",
            "example": "Ops room"
          },
          "failure_count": {
            "type": "integer",
            "format": "int64",
            "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
            "example": 0
          },
          "paused_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When repeated failures paused it (RFC 3339); `null` while active.",
            "example": "2026-11-21T10:00:00+00:00"
          },
          "paused_reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why it was paused.",
            "example": "unauthenticated: the webhook was deleted"
          },
          "created_at": {
            "type": "string",
            "description": "When it was made (RFC 3339).",
            "example": "2026-11-20T09:00:00+00:00"
          }
        }
      },
      "description": "Oldest first."
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/notifications/channels`

Makes a library channel on a notifier instance.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`NewChannelBody`](schemas.md#newchannelbody)

- `plugin_instance_id` *string*, **required** — A notifier instance that sends to channels; its own settings say where (a webhook URL, a service key).
- `label` *string*, **required** — Name shown in the routing matrix, 1 to 128 bytes on one line.

### Responses

- `201` — The channel. Body: [`DestinationJson`](schemas.md#destinationjson) (`application/json`)
  - `id` *string*, **required** — Destination id.
  - `kind` *[`DestinationKind`](schemas.md#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
  - `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
  - `label` *string*, **required** — Name shown in pickers.
  - `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
  - `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
  - `paused_reason` *string or null* — Why it was paused.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `400` — A bad label, or an instance that is not a channel notifier. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The channel could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "label": "Ops room"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "label": "Ops room"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/notifications/channels",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "label": "Ops room"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "label": "Ops room"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/notifications/channels")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "label": "Ops room"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `201`: The channel.

```json
{
  "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
  "kind": "channel",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "label": "Ops room",
  "failure_count": 0,
  "paused_at": "2026-11-21T10:00:00+00:00",
  "paused_reason": "unauthenticated: the webhook was deleted",
  "created_at": "2026-11-20T09:00:00+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
  "required": [
    "id",
    "kind",
    "plugin_instance_id",
    "label",
    "failure_count",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Destination id.",
      "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
    },
    "kind": {
      "type": "string",
      "description": "`channel` (the library's) or `personal` (a patron's).",
      "enum": [
        "channel",
        "personal"
      ]
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "The notifier instance that carries it.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "label": {
      "type": "string",
      "description": "Name shown in pickers.",
      "example": "Ops room"
    },
    "failure_count": {
      "type": "integer",
      "format": "int64",
      "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
      "example": 0
    },
    "paused_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When repeated failures paused it (RFC 3339); `null` while active.",
      "example": "2026-11-21T10:00:00+00:00"
    },
    "paused_reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it was paused.",
      "example": "unauthenticated: the webhook was deleted"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-20T09:00:00+00:00"
    }
  }
}
```

Status `400`: A bad label, or an instance that is not a channel notifier. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "plugin instance 3f9c2b7e does not send to channels"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `plugin_instance_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The channel could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `DELETE /api/notifications/channels/{id}`

Deletes a channel with its routes and delivery-log rows.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Channel id.

### Responses

- `204` — The channel is gone. No body.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such channel. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The channel could not be deleted. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/notifications/channels/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/notifications/channels/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The channel is gone. No body.

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

Status `404`: No such channel.

```json
{
  "error": "channel 8a7b6c5d not found"
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

Status `500`: The channel could not be deleted.

```json
{
  "error": "the library could not be read or changed"
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

## `PATCH /api/notifications/channels/{id}`

Renames a channel.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Channel id.

### Request body

`application/json`: [`RenameBody`](schemas.md#renamebody)

- `label` *string*, **required** — The new name, 1 to 128 bytes on one line.

### Responses

- `200` — The channel. Body: [`DestinationJson`](schemas.md#destinationjson) (`application/json`)
  - `id` *string*, **required** — Destination id.
  - `kind` *[`DestinationKind`](schemas.md#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
  - `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
  - `label` *string*, **required** — Name shown in pickers.
  - `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
  - `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
  - `paused_reason` *string or null* — Why it was paused.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `400` — A bad label. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such channel. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The channel could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels/<id>' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"label": "Ops room"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels/<id>' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"label": "Ops room"}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/notifications/channels/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"label": "Ops room"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels/<id>", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"label": "Ops room"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/notifications/channels/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"label": "Ops room"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The channel.

```json
{
  "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
  "kind": "channel",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "label": "Ops room",
  "failure_count": 0,
  "paused_at": "2026-11-21T10:00:00+00:00",
  "paused_reason": "unauthenticated: the webhook was deleted",
  "created_at": "2026-11-20T09:00:00+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
  "required": [
    "id",
    "kind",
    "plugin_instance_id",
    "label",
    "failure_count",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Destination id.",
      "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
    },
    "kind": {
      "type": "string",
      "description": "`channel` (the library's) or `personal` (a patron's).",
      "enum": [
        "channel",
        "personal"
      ]
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "The notifier instance that carries it.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "label": {
      "type": "string",
      "description": "Name shown in pickers.",
      "example": "Ops room"
    },
    "failure_count": {
      "type": "integer",
      "format": "int64",
      "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
      "example": 0
    },
    "paused_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When repeated failures paused it (RFC 3339); `null` while active.",
      "example": "2026-11-21T10:00:00+00:00"
    },
    "paused_reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it was paused.",
      "example": "unauthenticated: the webhook was deleted"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-20T09:00:00+00:00"
    }
  }
}
```

Status `400`: A bad label. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "a destination label is one line of 1 to 128 bytes"
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

Status `404`: No such channel.

```json
{
  "error": "channel 8a7b6c5d not found"
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
Failed to deserialize the JSON body into the target type: missing field `label` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The channel could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/notifications/channels/{id}/resume`

Resumes a channel repeated failures paused, and clears its failure
count.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Channel id.

### Responses

- `200` — The channel. Body: [`DestinationJson`](schemas.md#destinationjson) (`application/json`)
  - `id` *string*, **required** — Destination id.
  - `kind` *[`DestinationKind`](schemas.md#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
  - `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
  - `label` *string*, **required** — Name shown in pickers.
  - `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
  - `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
  - `paused_reason` *string or null* — Why it was paused.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such channel. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The channel could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels/<id>/resume' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels/<id>/resume' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/notifications/channels/<id>/resume",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels/<id>/resume", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/notifications/channels/<id>/resume")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The channel.

```json
{
  "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
  "kind": "channel",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "label": "Ops room",
  "failure_count": 0,
  "paused_at": "2026-11-21T10:00:00+00:00",
  "paused_reason": "unauthenticated: the webhook was deleted",
  "created_at": "2026-11-20T09:00:00+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
  "required": [
    "id",
    "kind",
    "plugin_instance_id",
    "label",
    "failure_count",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Destination id.",
      "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
    },
    "kind": {
      "type": "string",
      "description": "`channel` (the library's) or `personal` (a patron's).",
      "enum": [
        "channel",
        "personal"
      ]
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "The notifier instance that carries it.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "label": {
      "type": "string",
      "description": "Name shown in pickers.",
      "example": "Ops room"
    },
    "failure_count": {
      "type": "integer",
      "format": "int64",
      "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
      "example": 0
    },
    "paused_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When repeated failures paused it (RFC 3339); `null` while active.",
      "example": "2026-11-21T10:00:00+00:00"
    },
    "paused_reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it was paused.",
      "example": "unauthenticated: the webhook was deleted"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-20T09:00:00+00:00"
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

Status `404`: No such channel.

```json
{
  "error": "channel 8a7b6c5d not found"
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

Status `500`: The channel could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/notifications/channels/{id}/test`

Sends a test notification to one channel, whatever the routes say.

The send runs as a job; its outcome lands in the delivery log.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `id` (path) *string*, **required** — Channel id.

### Responses

- `202` — The test is accepted. Its send job is queued, or, when the queue is full or enqueueing fails, may be queued by a later sweep. Body: [`TestSent`](schemas.md#testsent) (`application/json`)
  - `notification_id` *string*, **required** — The stored test notification.
  - `delivery_id` *string*, **required** — Its delivery; follow it in the delivery log.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such channel. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The channel is paused; resume it first. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The test could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/channels/<id>/test' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/channels/<id>/test' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/notifications/channels/<id>/test",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/channels/<id>/test", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/notifications/channels/<id>/test")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `202`: The test is accepted. Its send job is queued, or, when the queue is full or enqueueing fails, may be queued by a later sweep.

```json
{
  "notification_id": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e",
  "delivery_id": "9e8d7c6b-5a4f-4e3d-8c2b-1a0f9e8d7c6b"
}
```

Schema:

```json
{
  "type": "object",
  "description": "The answer to a test send.",
  "required": [
    "notification_id",
    "delivery_id"
  ],
  "properties": {
    "notification_id": {
      "type": "string",
      "description": "The stored test notification.",
      "example": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e"
    },
    "delivery_id": {
      "type": "string",
      "description": "Its delivery; follow it in the delivery log.",
      "example": "9e8d7c6b-5a4f-4e3d-8c2b-1a0f9e8d7c6b"
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

Status `404`: No such channel.

```json
{
  "error": "channel 8a7b6c5d not found"
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

Status `409`: The channel is paused; resume it first.

```json
{
  "error": "the channel is paused; resume it before testing it"
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

Status `500`: The test could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/notifications/routes`

Lists every routing rule.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The routes, by type. Body: [`RouteTable`](schemas.md#routetable) (`application/json`)
  - `routes` *map of string to array of [`RouteJson`](schemas.md#routejson)*, **required** — Type id to its routes.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/routes' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/routes' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/notifications/routes",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/routes", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/notifications/routes")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The routes, by type.

```json
{
  "routes": {
    "test": [
      {
        "destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
        "min_severity": "info"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Every routing rule, by type.",
  "required": [
    "routes"
  ],
  "properties": {
    "routes": {
      "type": "object",
      "description": "Type id to its routes.",
      "additionalProperties": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "One route: a channel, or the built-in \"In app\" target, and the least\nseverity it takes.",
          "required": [
            "destination_id",
            "min_severity"
          ],
          "properties": {
            "destination_id": {
              "type": "string",
              "description": "The channel's id, or `in-app` for the built-in target that puts the\nnotification in every active Owner's and Administrator's inbox\n(the bell in the web app, and browser or desktop notifications where\nthey turned them on). `in-app` runs no plugin and is listed first.",
              "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
            },
            "min_severity": {
              "type": "string",
              "description": "`info`, `warning` or `critical`: notifications less urgent than\nthis skip the channel.",
              "example": "warning"
            }
          },
          "additionalProperties": false
        }
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "test": [
          {
            "destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
            "min_severity": "info"
          }
        ]
      }
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `PUT /api/notifications/routes/{type}`

Replaces the routes of one operational type.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `type` (path) *string*, **required** — Notification type id.

### Request body

`application/json`: [`RoutesBody`](schemas.md#routesbody)

- `routes` *array of [`RouteJson`](schemas.md#routejson)*, **required** — The type's routes, replacing the old ones; at most 32. Empty sends the type nowhere. At most 32 items.

### Responses

- `204` — The routes are replaced. No body.
- `400` — An unknown or patron type (either may still be cleared with no routes, to remove rules left from before the type changed), a bad severity, a personal destination, a repeat, or more than 32. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — A destination does not exist. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The routes could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/routes/<type>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"routes": [{"destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d", "min_severity": "warning"}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/routes/<type>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"routes": [{"destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d", "min_severity": "warning"}]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/notifications/routes/<type>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"routes": [{"destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d", "min_severity": "warning"}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/routes/<type>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"routes": [{"destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d", "min_severity": "warning"}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/notifications/routes/<type>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"routes": [{"destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d", "min_severity": "warning"}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The routes are replaced. No body.

Status `400`: An unknown or patron type (either may still be cleared with no routes, to remove rules left from before the type changed), a bad severity, a personal destination, a repeat, or more than 32. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "destination 8a7b6c5d is routed twice"
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

Status `404`: A destination does not exist.

```json
{
  "error": "notification destination 8a7b6c5d not found"
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
Failed to deserialize the JSON body into the target type: missing field `routes` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The routes could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/notifications/offers`

Lists which notifiers patrons may pick for each patron type.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The offers, by type. Body: [`OfferTable`](schemas.md#offertable) (`application/json`)
  - `offers` *map of string to array of [`OfferJson`](schemas.md#offerjson)*, **required** — Type id to its offers.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/offers' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/offers' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/notifications/offers",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/offers", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/notifications/offers")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The offers, by type.

```json
{
  "offers": {
    "discover:new_book": [
      {
        "default_on": true,
        "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
      }
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Every patron offer, by type.",
  "required": [
    "offers"
  ],
  "properties": {
    "offers": {
      "type": "object",
      "description": "Type id to its offers.",
      "additionalProperties": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "One notifier offered to patrons for a type.",
          "required": [
            "plugin_instance_id",
            "default_on"
          ],
          "properties": {
            "plugin_instance_id": {
              "type": "string",
              "description": "A notifier instance that sends to personal destinations.",
              "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
            },
            "default_on": {
              "type": "boolean",
              "description": "Whether a patron who chose nothing gets the type there.",
              "example": true
            }
          },
          "additionalProperties": false
        }
      },
      "propertyNames": {
        "type": "string"
      },
      "example": {
        "discover:new_book": [
          {
            "default_on": true,
            "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          }
        ]
      }
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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `PUT /api/notifications/offers/{type}`

Replaces which notifiers patrons may pick for one patron type.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `type` (path) *string*, **required** — Notification type id.

### Request body

`application/json`: [`OffersBody`](schemas.md#offersbody)

- `offers` *array of [`OfferJson`](schemas.md#offerjson)*, **required** — The type's offers, replacing the old ones; at most 32. Empty keeps the type in patrons' inboxes only. At most 32 items.

### Responses

- `204` — The offers are replaced. No body.
- `400` — An unknown or operational type (either may still be cleared with no offers, to remove offers left from before the type changed), an instance that is not a personal notifier, a repeat, or more than 32. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The offers could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/offers/<type>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"offers": [{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "default_on": true}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/offers/<type>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"offers": [{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "default_on": true}]}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/notifications/offers/<type>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"offers": [{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "default_on": True}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/offers/<type>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"offers": [{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "default_on": true}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/notifications/offers/<type>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"offers": [{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "default_on": true}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The offers are replaced. No body.

Status `400`: An unknown or operational type (either may still be cleared with no offers, to remove offers left from before the type changed), an instance that is not a personal notifier, a repeat, or more than 32. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "plugin instance 3f9c2b7e does not send to patrons"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `offers` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The offers could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/notifications/deliveries`

Pages the delivery log, newest first.

Access: the operator, an Owner or an Administrator.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `destination_id` (query) *string* — Only this destination's deliveries.
- `cursor` (query) *string* — `next_cursor` of the previous page.
- `limit` (query) *integer (int64)* — Page size (default 50): a whole number, where 0 is read as 1 and anything from 201 up to 18446744073709551615 as 200; a negative or non-numeric value, or one past 18446744073709551615, is refused with a plain-text 400. A page may hold fewer rows than asked when the library's database returns fewer per read.

### Responses

- `200` — One page of deliveries. Body: [`DeliveryPage`](schemas.md#deliverypage) (`application/json`)
  - `deliveries` *array of [`DeliveryJson`](schemas.md#deliveryjson)*, **required** — Newest first.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; `null` on the last page, where paging stops.
- `400` — The cursor is not a valid cursor; pass back this route's own `next_cursor` (JSON). Or `limit` is not a whole number that fits in 64 bits (plain text, from the query parser). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The caller is not an Administrator. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/notifications/deliveries' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/notifications/deliveries' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/notifications/deliveries",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/notifications/deliveries", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/notifications/deliveries")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One page of deliveries.

```json
{
  "deliveries": [
    {
      "id": "9e8d7c6b-5a4f-4e3d-8c2b-1a0f9e8d7c6b",
      "notification_id": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e",
      "notification_type": "test",
      "title": "Test notification",
      "destination_id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "state": "delivered",
      "attempts": 1,
      "provider_id": "1700000000.000200",
      "reason": "rate_limited: slow down",
      "created_at": "2026-11-21T10:00:00+00:00",
      "updated_at": "2026-11-21T10:00:02+00:00"
    }
  ],
  "next_cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNi0xMS0yMVQxMDowMDowMCswMDowMCIsImlkIjoiOWU4ZCJ9"
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of the delivery log.",
  "required": [
    "deliveries"
  ],
  "properties": {
    "deliveries": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One delivery-log row.",
        "required": [
          "id",
          "notification_id",
          "notification_type",
          "title",
          "destination_id",
          "plugin_instance_id",
          "state",
          "attempts",
          "created_at",
          "updated_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Delivery id.",
            "example": "9e8d7c6b-5a4f-4e3d-8c2b-1a0f9e8d7c6b"
          },
          "notification_id": {
            "type": "string",
            "description": "The notification.",
            "example": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e"
          },
          "notification_type": {
            "type": "string",
            "description": "Its type.",
            "example": "test"
          },
          "title": {
            "type": "string",
            "description": "Its title.",
            "example": "Test notification"
          },
          "destination_id": {
            "type": "string",
            "description": "The destination.",
            "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
          },
          "plugin_instance_id": {
            "type": "string",
            "description": "The notifier instance.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "state": {
            "type": "string",
            "description": "`pending`, `delivered`, `failed` or `dropped`.",
            "example": "delivered"
          },
          "attempts": {
            "type": "integer",
            "format": "int64",
            "description": "Send attempts made.",
            "example": 1
          },
          "provider_id": {
            "type": [
              "string",
              "null"
            ],
            "description": "The service's id for the delivered message.",
            "example": "1700000000.000200"
          },
          "reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "The last failure or retry reason.",
            "example": "rate_limited: slow down"
          },
          "created_at": {
            "type": "string",
            "description": "When it was made (RFC 3339).",
            "example": "2026-11-21T10:00:00+00:00"
          },
          "updated_at": {
            "type": "string",
            "description": "When it last changed (RFC 3339).",
            "example": "2026-11-21T10:00:02+00:00"
          }
        }
      },
      "description": "Newest first."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; `null` on the last page, where\npaging stops.",
      "example": "eyJjcmVhdGVkX2F0IjoiMjAyNi0xMS0yMVQxMDowMDowMCswMDowMCIsImlkIjoiOWU4ZCJ9"
    }
  }
}
```

Status `400`: The cursor is not a valid cursor; pass back this route's own `next_cursor` (JSON). Or `limit` is not a whole number that fits in 64 bits (plain text, from the query parser).

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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/me/notifications`

Reads the caller's notification settings: the types they can receive
beyond the inbox, what they chose, and their destinations.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Responses

- `200` — The settings. Body: [`PatronSettings`](schemas.md#patronsettings) (`application/json`)
  - `types` *array of [`PatronTypeJson`](schemas.md#patrontypejson)*, **required** — Patron types administrators offer a notifier for.
  - `destinations` *array of [`DestinationJson`](schemas.md#destinationjson)*, **required** — The patron's own destinations.
  - `notifiers` *array of [`NotifierJson`](schemas.md#notifierjson)*, **required** — Notifiers the patron can add a destination on.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/me/notifications",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/me/notifications")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings.

```json
{
  "types": [
    {
      "id": "discover:new_book",
      "title": "A new book you might like",
      "offers": [
        {
          "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
          "name": "Telegram",
          "default_on": true
        }
      ],
      "chosen": false,
      "enabled": true,
      "destination_ids": []
    }
  ],
  "destinations": [
    {
      "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
      "kind": "channel",
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "label": "Ops room",
      "failure_count": 0,
      "paused_at": "2026-11-21T10:00:00+00:00",
      "paused_reason": "unauthenticated: the webhook was deleted",
      "created_at": "2026-11-20T09:00:00+00:00"
    }
  ],
  "notifiers": [
    {
      "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
      "plugin_key": "plugin:telegram",
      "name": "Telegram",
      "channel": false,
      "personal": true,
      "address_fields": [
        {
          "key": "chat_id",
          "label": "Chat id",
          "required": true,
          "secret": false
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
  "description": "A patron's notification settings.",
  "required": [
    "types",
    "destinations",
    "notifiers"
  ],
  "properties": {
    "types": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One patron type and what the patron chose for it.",
        "required": [
          "id",
          "title",
          "offers",
          "chosen",
          "enabled",
          "destination_ids"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Type id.",
            "example": "discover:new_book"
          },
          "title": {
            "type": "string",
            "description": "Title.",
            "example": "A new book you might like"
          },
          "offers": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One notifier a patron may pick for a type.",
              "required": [
                "plugin_instance_id",
                "name",
                "default_on"
              ],
              "properties": {
                "plugin_instance_id": {
                  "type": "string",
                  "description": "The notifier instance.",
                  "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
                },
                "name": {
                  "type": "string",
                  "description": "Its name.",
                  "example": "Telegram"
                },
                "default_on": {
                  "type": "boolean",
                  "description": "Whether the type goes there when the patron chose nothing.",
                  "example": true
                }
              }
            },
            "description": "Notifiers offered for it."
          },
          "chosen": {
            "type": "boolean",
            "description": "Whether the patron made a choice; without one the defaults apply.",
            "example": false
          },
          "enabled": {
            "type": "boolean",
            "description": "The patron's preference, not a delivery result: `false` keeps the\ntype in the inbox only. `true` when nothing is chosen, in which case\nthe offers' defaults pick where it goes.",
            "example": true
          },
          "destination_ids": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "The destinations the patron saved for it, when chosen. These are\nsaved choices, not live targets: routing skips one whose notifier is\nno longer offered for the type, so check `offers` before showing one\nas eligible or sending the list back.",
            "example": []
          }
        }
      },
      "description": "Patron types administrators offer a notifier for."
    },
    "destinations": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
        "required": [
          "id",
          "kind",
          "plugin_instance_id",
          "label",
          "failure_count",
          "created_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Destination id.",
            "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
          },
          "kind": {
            "type": "string",
            "description": "`channel` (the library's) or `personal` (a patron's).",
            "enum": [
              "channel",
              "personal"
            ]
          },
          "plugin_instance_id": {
            "type": "string",
            "description": "The notifier instance that carries it.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "label": {
            "type": "string",
            "description": "Name shown in pickers.",
            "example": "Ops room"
          },
          "failure_count": {
            "type": "integer",
            "format": "int64",
            "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
            "example": 0
          },
          "paused_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When repeated failures paused it (RFC 3339); `null` while active.",
            "example": "2026-11-21T10:00:00+00:00"
          },
          "paused_reason": {
            "type": [
              "string",
              "null"
            ],
            "description": "Why it was paused.",
            "example": "unauthenticated: the webhook was deleted"
          },
          "created_at": {
            "type": "string",
            "description": "When it was made (RFC 3339).",
            "example": "2026-11-20T09:00:00+00:00"
          }
        }
      },
      "description": "The patron's own destinations."
    },
    "notifiers": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One notifier instance.",
        "required": [
          "plugin_instance_id",
          "plugin_key",
          "name",
          "channel",
          "personal",
          "address_fields"
        ],
        "properties": {
          "plugin_instance_id": {
            "type": "string",
            "description": "Plugin instance id.",
            "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
          },
          "plugin_key": {
            "type": "string",
            "description": "Canonical plugin key.",
            "example": "plugin:telegram"
          },
          "name": {
            "type": "string",
            "description": "Name shown in pickers.",
            "example": "Telegram"
          },
          "channel": {
            "type": "boolean",
            "description": "Whether administrators can make library channels on it.",
            "example": false
          },
          "personal": {
            "type": "boolean",
            "description": "Whether patrons can add their own address on it.",
            "example": true
          },
          "address_fields": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One field a patron fills in for a personal destination.",
              "required": [
                "key",
                "label",
                "required",
                "secret"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Key in the address document.",
                  "example": "chat_id"
                },
                "label": {
                  "type": "string",
                  "description": "Label shown on the form.",
                  "example": "Chat id"
                },
                "required": {
                  "type": "boolean",
                  "description": "Whether a destination must carry it.",
                  "example": true
                },
                "secret": {
                  "type": "boolean",
                  "description": "Whether the value is a secret: shown masked, never logged.",
                  "example": false
                }
              }
            },
            "description": "What a patron enters for a personal destination."
          }
        }
      },
      "description": "Notifiers the patron can add a destination on."
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `PUT /api/me/notifications/choices/{type}`

Sets what the caller gets for one patron type, replacing the defaults.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `type` (path) *string*, **required** — Notification type id.

### Request body

`application/json`: [`ChoiceBody`](schemas.md#choicebody)

- `enabled` *boolean*, **required** — `false` keeps the type in the inbox only.
- `destination_ids` *array of string* — The caller's own destinations it goes to; at most 32. Left out or empty, the type goes to no destination outside the inbox, even with `enabled: true`; delete the choice to return to the defaults. At most 32 items.

### Responses

- `204` — The choice is saved. No body.
- `400` — A bad type id, a repeat, more than 32 destinations, or (when enabled) a destination on a notifier not offered for the type. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — A destination is not one of the caller's. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The choice could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/choices/<type>' \
    -X PUT \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"enabled": true}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/choices/<type>' `
    -Method Put `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"enabled": true}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/me/notifications/choices/<type>",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"enabled": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/choices/<type>", {
  method: "PUT",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"enabled": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/me/notifications/choices/<type>")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"enabled": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The choice is saved. No body.

Status `400`: A bad type id, a repeat, more than 32 destinations, or (when enabled) a destination on a notifier not offered for the type. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "destination 8a7b6c5d is chosen twice"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: A destination is not one of the caller's.

```json
{
  "error": "notification destination 8a7b6c5d not found"
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
Failed to deserialize the JSON body into the target type: missing field `enabled` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The choice could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `DELETE /api/me/notifications/choices/{type}`

Drops the caller's choice for one type, so the defaults apply again.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `type` (path) *string*, **required** — Notification type id.

### Responses

- `204` — The defaults apply again. No body.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The choice could not be removed. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/choices/<type>' \
    -X DELETE \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/choices/<type>' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/me/notifications/choices/<type>",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/choices/<type>", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/me/notifications/choices/<type>")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The defaults apply again. No body.

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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `500`: The choice could not be removed.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/me/notifications/destinations`

Adds a personal destination for the caller.

The notifier normalises and checks the address first
(`Notifier.validateDestination`); the host checks its answer again
against the declared fields before saving it.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Request body

`application/json`: [`NewPersonalBody`](schemas.md#newpersonalbody)

- `plugin_instance_id` *string*, **required** — A notifier instance that sends to patrons.
- `address` *map of string to string*, **required** — The address, as the notifier's `address_fields` describe it: a flat object of strings. A key written twice is refused (`422`) rather than one value kept.
- `label` *string or null* — Name shown in pickers; the notifier's own label for the address, or the notifier's name, when absent.

### Responses

- `201` — The destination. Body: [`DestinationJson`](schemas.md#destinationjson) (`application/json`)
  - `id` *string*, **required** — Destination id.
  - `kind` *[`DestinationKind`](schemas.md#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
  - `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
  - `label` *string*, **required** — Name shown in pickers.
  - `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
  - `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
  - `paused_reason` *string or null* — Why it was paused.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `400` — A bad address or label, an instance that does not send to patrons or is not offered for any type, or a patron who already keeps 32 addresses. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `409` — The notifier is not running on this server. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The destination could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `502` — The notifier could not be reached, or the host could not use its reply (for example a normalised address missing a required field). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/destinations' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "address": {"chat_id": "123456789"}}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/destinations' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "address": {"chat_id": "123456789"}}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/me/notifications/destinations",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "address": {"chat_id": "123456789"}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/destinations", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "address": {"chat_id": "123456789"}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/me/notifications/destinations")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b", "address": {"chat_id": "123456789"}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `201`: The destination.

```json
{
  "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
  "kind": "channel",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "label": "Ops room",
  "failure_count": 0,
  "paused_at": "2026-11-21T10:00:00+00:00",
  "paused_reason": "unauthenticated: the webhook was deleted",
  "created_at": "2026-11-20T09:00:00+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
  "required": [
    "id",
    "kind",
    "plugin_instance_id",
    "label",
    "failure_count",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Destination id.",
      "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
    },
    "kind": {
      "type": "string",
      "description": "`channel` (the library's) or `personal` (a patron's).",
      "enum": [
        "channel",
        "personal"
      ]
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "The notifier instance that carries it.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "label": {
      "type": "string",
      "description": "Name shown in pickers.",
      "example": "Ops room"
    },
    "failure_count": {
      "type": "integer",
      "format": "int64",
      "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
      "example": 0
    },
    "paused_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When repeated failures paused it (RFC 3339); `null` while active.",
      "example": "2026-11-21T10:00:00+00:00"
    },
    "paused_reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it was paused.",
      "example": "unauthenticated: the webhook was deleted"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-20T09:00:00+00:00"
    }
  }
}
```

Status `400`: A bad address or label, an instance that does not send to patrons or is not offered for any type, or a patron who already keeps 32 addresses. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "address field `chat_id` is required"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `409`: The notifier is not running on this server.

```json
{
  "error": "the notifier is not running on this server"
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
Failed to deserialize the JSON body into the target type: missing field `plugin_instance_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The destination could not be written.

```json
{
  "error": "the library could not be read or changed"
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

Status `502`: The notifier could not be reached, or the host could not use its reply (for example a normalised address missing a required field).

```json
{
  "error": "the notifier could not check the address"
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

## `DELETE /api/me/notifications/destinations/{id}`

Deletes one of the caller's destinations.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Destination id.

### Responses

- `204` — The destination is gone. No body.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — Not one of the caller's destinations. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The destination could not be deleted. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/destinations/<id>' \
    -X DELETE \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/destinations/<id>' `
    -Method Delete `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/me/notifications/destinations/<id>",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/destinations/<id>", {
  method: "DELETE",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/me/notifications/destinations/<id>")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The destination is gone. No body.

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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: Not one of the caller's destinations.

```json
{
  "error": "destination 8a7b6c5d not found"
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

Status `500`: The destination could not be deleted.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/me/notifications/destinations/{id}/resume`

Resumes one of the caller's destinations that repeated failures paused,
and clears its failure count.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Destination id.

### Responses

- `200` — The destination. Body: [`DestinationJson`](schemas.md#destinationjson) (`application/json`)
  - `id` *string*, **required** — Destination id.
  - `kind` *[`DestinationKind`](schemas.md#destinationkind)*, **required** — `channel` (the library's) or `personal` (a patron's).
  - `plugin_instance_id` *string*, **required** — The notifier instance that carries it.
  - `label` *string*, **required** — Name shown in pickers.
  - `failure_count` *integer (int64)*, **required** — Deliveries in a row that ended `failed`, whether refused for good or out of attempts; a delivery, or resuming the destination, resets it to 0. It pauses at 5.
  - `paused_at` *string or null* — When repeated failures paused it (RFC 3339); `null` while active.
  - `paused_reason` *string or null* — Why it was paused.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — Not one of the caller's destinations. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The destination could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/destinations/<id>/resume' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/destinations/<id>/resume' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/me/notifications/destinations/<id>/resume",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/destinations/<id>/resume", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/me/notifications/destinations/<id>/resume")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The destination.

```json
{
  "id": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d",
  "kind": "channel",
  "plugin_instance_id": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b",
  "label": "Ops room",
  "failure_count": 0,
  "paused_at": "2026-11-21T10:00:00+00:00",
  "paused_reason": "unauthenticated: the webhook was deleted",
  "created_at": "2026-11-20T09:00:00+00:00"
}
```

Schema:

```json
{
  "type": "object",
  "description": "A destination as administrators and its owner see it. The address is\nnever returned: it may hold secrets.",
  "required": [
    "id",
    "kind",
    "plugin_instance_id",
    "label",
    "failure_count",
    "created_at"
  ],
  "properties": {
    "id": {
      "type": "string",
      "description": "Destination id.",
      "example": "8a7b6c5d-4e3f-4a2b-9c1d-0e9f8a7b6c5d"
    },
    "kind": {
      "type": "string",
      "description": "`channel` (the library's) or `personal` (a patron's).",
      "enum": [
        "channel",
        "personal"
      ]
    },
    "plugin_instance_id": {
      "type": "string",
      "description": "The notifier instance that carries it.",
      "example": "3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b"
    },
    "label": {
      "type": "string",
      "description": "Name shown in pickers.",
      "example": "Ops room"
    },
    "failure_count": {
      "type": "integer",
      "format": "int64",
      "description": "Deliveries in a row that ended `failed`, whether refused for good\nor out of attempts; a delivery, or resuming the destination, resets\nit to 0. It pauses at 5.",
      "example": 0
    },
    "paused_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "When repeated failures paused it (RFC 3339); `null` while active.",
      "example": "2026-11-21T10:00:00+00:00"
    },
    "paused_reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why it was paused.",
      "example": "unauthenticated: the webhook was deleted"
    },
    "created_at": {
      "type": "string",
      "description": "When it was made (RFC 3339).",
      "example": "2026-11-20T09:00:00+00:00"
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: Not one of the caller's destinations.

```json
{
  "error": "destination 8a7b6c5d not found"
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

Status `500`: The destination could not be written.

```json
{
  "error": "the library could not be read or changed"
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

## `GET /api/me/notifications/inbox`

Pages the caller's inbox, newest first.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Parameters

- `cursor` (query) *string* — `next_cursor` of the previous page.
- `limit` (query) *integer (int64)* — Page size (default 50): a whole number, where 0 is read as 1 and anything from 201 up to 18446744073709551615 as 200; a negative or non-numeric value, or one past 18446744073709551615, is refused with a plain-text 400. A page may hold fewer rows than asked when the library's database returns fewer per read.

### Responses

- `200` — One page of the inbox. Body: [`InboxPage`](schemas.md#inboxpage) (`application/json`)
  - `entries` *array of [`InboxEntryJson`](schemas.md#inboxentryjson)*, **required** — Newest first.
  - `unread` *integer (int64)*, **required** — Unread entries in the whole inbox. At least 0.
  - `next_cursor` *string or null* — Pass as `cursor` for the next page; `null` on the last page, where paging stops.
- `400` — The cursor is not a valid cursor; pass back this route's own `next_cursor` (JSON). Or `limit` is not a whole number that fits in 64 bits (plain text, from the query parser). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/inbox' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/inbox' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/me/notifications/inbox",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/inbox", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/me/notifications/inbox")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One page of the inbox.

```json
{
  "entries": [
    {
      "id": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e",
      "notification_type": "discover:new_book",
      "severity": "info",
      "title": "A new book you might like",
      "text": "The Way of Kings is on Libro.fm.",
      "link_path": "/works/6c1f2e8a",
      "fields": [
        {
          "name": "Narrator",
          "value": "Kate Reading"
        }
      ],
      "created_at": "2026-11-21T10:00:00+00:00",
      "read_at": "2026-11-21T10:05:00+00:00"
    }
  ],
  "unread": 3,
  "next_cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNi0xMS0yMVQxMDowMDowMCswMDowMCIsImlkIjoiMWIyYyJ9"
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of the inbox.",
  "required": [
    "entries",
    "unread"
  ],
  "properties": {
    "entries": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One inbox entry.",
        "required": [
          "id",
          "notification_type",
          "severity",
          "title",
          "text",
          "fields",
          "created_at"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Notification id.",
            "example": "1b2c3d4e-5f6a-4b7c-8d9e-0f1a2b3c4d5e"
          },
          "notification_type": {
            "type": "string",
            "description": "Type id.",
            "example": "discover:new_book"
          },
          "severity": {
            "type": "string",
            "description": "`info`, `warning` or `critical`.",
            "example": "info"
          },
          "title": {
            "type": "string",
            "description": "Title.",
            "example": "A new book you might like"
          },
          "text": {
            "type": "string",
            "description": "Plain-text body.",
            "example": "The Way of Kings is on Libro.fm."
          },
          "link_path": {
            "type": [
              "string",
              "null"
            ],
            "description": "Path inside the library it links to.",
            "example": "/works/6c1f2e8a"
          },
          "fields": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One labelled value.",
              "required": [
                "name",
                "value"
              ],
              "properties": {
                "name": {
                  "type": "string",
                  "description": "Label.",
                  "example": "Narrator"
                },
                "value": {
                  "type": "string",
                  "description": "Value.",
                  "example": "Kate Reading"
                }
              }
            },
            "description": "Labelled values."
          },
          "created_at": {
            "type": "string",
            "description": "When it arrived (RFC 3339).",
            "example": "2026-11-21T10:00:00+00:00"
          },
          "read_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When the patron read it (RFC 3339); `null` while unread.",
            "example": "2026-11-21T10:05:00+00:00"
          }
        }
      },
      "description": "Newest first."
    },
    "unread": {
      "type": "integer",
      "format": "int64",
      "description": "Unread entries in the whole inbox.",
      "example": 3,
      "minimum": 0
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Pass as `cursor` for the next page; `null` on the last page, where\npaging stops.",
      "example": "eyJjcmVhdGVkX2F0IjoiMjAyNi0xMS0yMVQxMDowMDowMCswMDowMCIsImlkIjoiMWIyYyJ9"
    }
  }
}
```

Status `400`: The cursor is not a valid cursor; pass back this route's own `next_cursor` (JSON). Or `limit` is not a whole number that fits in 64 bits (plain text, from the query parser).

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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`.

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

Status `500`: The library could not be read.

```json
{
  "error": "the library could not be read or changed"
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

## `POST /api/me/notifications/inbox/read`

Marks one inbox entry, or all of them, read.

Access: any signed-in library user, for their own settings, or an
operator session acting as a library user; a plain operator session or
the operator token has none.

**Access:** The operator, or any signed-in user.

**Credentials:** `portalSession` or `operatorSession`

### Request body

`application/json`: [`ReadBody`](schemas.md#readbody)

- `notification_id` *string or null* — One notification to mark read; absent or `null` marks the whole inbox read.

### Responses

- `200` — What was marked. Body: [`ReadMarked`](schemas.md#readmarked) (`application/json`)
  - `marked` *integer (int64)*, **required** — Entries that were unread and are now read. At least 0.
- `400` — The body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — No one is signed in. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — The notification is not in the caller's inbox. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The inbox could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/me/notifications/inbox/read' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/me/notifications/inbox/read' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/me/notifications/inbox/read",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/me/notifications/inbox/read", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/me/notifications/inbox/read")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What was marked.

```json
{
  "marked": 1
}
```

Schema:

```json
{
  "type": "object",
  "description": "How many entries a read marked.",
  "required": [
    "marked"
  ],
  "properties": {
    "marked": {
      "type": "integer",
      "format": "int64",
      "description": "Entries that were unread and are now read.",
      "example": 1,
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

Status `403`: The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: The notification is not in the caller's inbox.

```json
{
  "error": "notification 1b2c3d4e not found"
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
Failed to deserialize the JSON body into the target type: notification_id: invalid type: integer `1`, expected a string at line 1 column 20
```

Schema:

A `text/plain` body: a string.

Status `500`: The inbox could not be written.

```json
{
  "error": "the library could not be read or changed"
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
