# Playback

Play sessions, audio streams, listening progress and bookmarks.

Back to the [web API reference](README.md).

## `POST /api/playback/sessions`

Opens a play session for the caller.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Request body

`application/json`: [`OpenSessionBody`](schemas.md#opensessionbody)

- `edition_id` *string*, **required** — The edition to play (1 to 128 bytes).
- `player` *[`PlayerKindParam`](schemas.md#playerkindparam)* — What kind of player asks; `web` when left out.
- `player_name` *string* — The player's own name, shown in listening history. Control characters are dropped and at most 64 bytes are kept.

### Responses

- `200` — The session, its tracks and where to start. Body: [`SessionJson`](schemas.md#sessionjson) (`application/json`)
  - `session_id` *string*, **required** — The session's id, for its sync, close and track routes.
  - `edition_id` *string*, **required** — The edition being played.
  - `work_id` *string or null* — Its work.
  - `position_ms` *integer (int64)*, **required** — Where to start, in milliseconds: the caller's stored position, or 0 for a first listen or an edition they had finished. At least 0.
  - `duration_ms` *integer (int64) or null* — Runtime in milliseconds when known (see the manifest). At least 0.
  - `tracks` *array of [`TrackJson`](schemas.md#trackjson)*, **required** — The tracks in play order, each with the path to stream it from.
  - `idle_secs` *integer (int64)*, **required** — Seconds without a report after which the session stops streaming and the player opens a new one.
- `400` — The edition id is empty or over 128 bytes. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition (`edition ... not found`), or no copy of it on this node that the caller may play (`not_playable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 4096 bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — This node's storage could not be asked; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/sessions' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/sessions' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/playback/sessions",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/sessions", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/playback/sessions")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The session, its tracks and where to start.

```json
{
  "session_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
  "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
  "position_ms": 754000,
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
  "idle_secs": 86400
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/playback/sessions` answer.",
  "required": [
    "session_id",
    "edition_id",
    "position_ms",
    "tracks",
    "idle_secs"
  ],
  "properties": {
    "session_id": {
      "type": "string",
      "description": "The session's id, for its sync, close and track routes.",
      "example": "7c9e6679-7425-40de-944b-e07fc1f90ae7"
    },
    "edition_id": {
      "type": "string",
      "description": "The edition being played.",
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
    "position_ms": {
      "type": "integer",
      "format": "int64",
      "description": "Where to start, in milliseconds: the caller's stored position, or\n0 for a first listen or an edition they had finished.",
      "example": 754000,
      "minimum": 0
    },
    "duration_ms": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in milliseconds when known (see the manifest).",
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
      "description": "The tracks in play order, each with the path to stream it from."
    },
    "idle_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds without a report after which the session stops streaming\nand the player opens a new one.",
      "example": 86400
    }
  }
}
```

Status `400`: The edition id is empty or over 128 bytes. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "edition_id must be 1 to 128 bytes"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

Status `404`: No such edition (`edition ... not found`), or no copy of it on this node that the caller may play (`not_playable`).

```json
{
  "error": "not_playable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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
Failed to deserialize the JSON body into the target type: missing field `edition_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

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

Status `503`: This node's storage could not be asked; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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

## `POST /api/playback/sessions/{id}/sync`

Reports where the caller is in a playing session.

Players report every few seconds while playing, and on pause and seek.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Session id from `POST /api/playback/sessions`.

### Request body

`application/json`: [`PlaybackSyncBody`](schemas.md#playbacksyncbody)

- `position_ms` *integer (int64)*, **required** — Position on the edition's timeline, in milliseconds from its start (at most sixty days). At least 0.
- `listened_ms` *integer (int64)*, **required** — Milliseconds listened since this session's last report, as the player counted them; left out reads as 0. At least 0.
- `duration_ms` *integer (int64) or null* — The edition's whole runtime in milliseconds, when the player knows it; left out or `null` keeps the stored one. At least 0.

### Responses

- `200` — The caller's progress after the report. Body: [`SyncJson`](schemas.md#syncjson) (`application/json`)
  - `progress` *[`ProgressJson`](schemas.md#progressjson)*, **required** — The caller's progress after the report.
  - `finished_now` *boolean*, **required** — True when this report is the one that reached the edition's end: the edition and its work are now finished for the caller.
- `400` — A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such session of the caller's (`unknown_session`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The session was already closed (`session_ended`); open a new one. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 1024 bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/playback/sessions/<id>/sync' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"position_ms": 754000, "listened_ms": 15000}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/sessions/<id>/sync' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"position_ms": 754000, "listened_ms": 15000}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/playback/sessions/<id>/sync",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"position_ms": 754000, "listened_ms": 15000},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/sessions/<id>/sync", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"position_ms": 754000, "listened_ms": 15000}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/playback/sessions/<id>/sync")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"position_ms": 754000, "listened_ms": 15000}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's progress after the report.

```json
{
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
  "finished_now": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/playback/sessions/{id}/sync` and `.../close` answer.",
  "required": [
    "progress",
    "finished_now"
  ],
  "properties": {
    "progress": {
      "type": "object",
      "description": "The caller's progress after the report.",
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
    "finished_now": {
      "type": "boolean",
      "description": "True when this report is the one that reached the edition's end:\nthe edition and its work are now finished for the caller.",
      "example": false
    }
  }
}
```

Status `400`: A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "position_ms must be at most 5184000000"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

Status `404`: No such session of the caller's (`unknown_session`).

```json
{
  "error": "unknown_session"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The session was already closed (`session_ended`); open a new one.

```json
{
  "error": "session_ended"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `413`: The body is over 1024 bytes.

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

## `POST /api/playback/sessions/{id}/close`

Sends a session's last report and ends it.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Session id from `POST /api/playback/sessions`.

### Request body

`application/json`: [`PlaybackSyncBody`](schemas.md#playbacksyncbody)

- `position_ms` *integer (int64)*, **required** — Position on the edition's timeline, in milliseconds from its start (at most sixty days). At least 0.
- `listened_ms` *integer (int64)*, **required** — Milliseconds listened since this session's last report, as the player counted them; left out reads as 0. At least 0.
- `duration_ms` *integer (int64) or null* — The edition's whole runtime in milliseconds, when the player knows it; left out or `null` keeps the stored one. At least 0.

### Responses

- `200` — The caller's progress after the report; the session is closed. Body: [`SyncJson`](schemas.md#syncjson) (`application/json`)
  - `progress` *[`ProgressJson`](schemas.md#progressjson)*, **required** — The caller's progress after the report.
  - `finished_now` *boolean*, **required** — True when this report is the one that reached the edition's end: the edition and its work are now finished for the caller.
- `400` — A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such session of the caller's (`unknown_session`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The session was already closed (`session_ended`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 1024 bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/playback/sessions/<id>/close' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"position_ms": 754000, "listened_ms": 15000}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/sessions/<id>/close' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"position_ms": 754000, "listened_ms": 15000}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/playback/sessions/<id>/close",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"position_ms": 754000, "listened_ms": 15000},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/sessions/<id>/close", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"position_ms": 754000, "listened_ms": 15000}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/playback/sessions/<id>/close")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"position_ms": 754000, "listened_ms": 15000}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's progress after the report; the session is closed.

```json
{
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
  "finished_now": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/playback/sessions/{id}/sync` and `.../close` answer.",
  "required": [
    "progress",
    "finished_now"
  ],
  "properties": {
    "progress": {
      "type": "object",
      "description": "The caller's progress after the report.",
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
    "finished_now": {
      "type": "boolean",
      "description": "True when this report is the one that reached the edition's end:\nthe edition and its work are now finished for the caller.",
      "example": false
    }
  }
}
```

Status `400`: A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "position_ms must be at most 5184000000"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

Status `404`: No such session of the caller's (`unknown_session`).

```json
{
  "error": "unknown_session"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The session was already closed (`session_ended`).

```json
{
  "error": "session_ended"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `413`: The body is over 1024 bytes.

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

## `GET /api/playback/sessions/{id}/tracks/{index}`

Streams one track of a play session.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Session id from `POST /api/playback/sessions`.
- `index` (path) *integer (int32)*, **required** — The track's `index` in the session's `tracks`.
- `Range` (header) *string or null* — One byte range (`bytes=A-B`, `bytes=A-` or `bytes=-N`); anything else reads the whole file.

### Responses

- `200` — The whole track. Body: [`BinaryBody`](schemas.md#binarybody) (`audio/mp4`); [`BinaryBody`](schemas.md#binarybody) (`audio/mpeg`); [`BinaryBody`](schemas.md#binarybody) (`audio/aac`); [`BinaryBody`](schemas.md#binarybody) (`audio/flac`); [`BinaryBody`](schemas.md#binarybody) (`audio/ogg`); [`BinaryBody`](schemas.md#binarybody) (`audio/wav`); [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `206` — The requested byte range, with `Content-Range`. Body: [`BinaryBody`](schemas.md#binarybody) (`audio/mp4`); [`BinaryBody`](schemas.md#binarybody) (`audio/mpeg`); [`BinaryBody`](schemas.md#binarybody) (`audio/aac`); [`BinaryBody`](schemas.md#binarybody) (`audio/flac`); [`BinaryBody`](schemas.md#binarybody) (`audio/ogg`); [`BinaryBody`](schemas.md#binarybody) (`audio/wav`); [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such session of the caller's (`unknown_session`), or the index names no track of it (`unknown_track`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The session no longer streams (`session_ended`): it was closed, sat a day without a report, the caller may no longer play the edition, or the edition's files changed. Open a new session. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `416` — The range starts at or past the end of the file (`range_not_satisfiable`); `Content-Range` states the size. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `421` — Another node opened this session (`misdirected`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — Storage is unavailable; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/sessions/<id>/tracks/<index>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/sessions/<id>/tracks/<index>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/playback/sessions/<id>/tracks/<index>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/sessions/<id>/tracks/<index>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/playback/sessions/<id>/tracks/<index>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The whole track. The body is binary (`audio/mp4`).

Schema:

A `audio/mp4` body: binary bytes.

Status `206`: The requested byte range, with `Content-Range`. The body is binary (`audio/mp4`).

Schema:

A `audio/mp4` body: binary bytes.

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

Status `404`: No such session of the caller's (`unknown_session`), or the index names no track of it (`unknown_track`).

```json
{
  "error": "unknown_track"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `409`: The session no longer streams (`session_ended`): it was closed, sat a day without a report, the caller may no longer play the edition, or the edition's files changed. Open a new session.

```json
{
  "error": "session_ended"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `416`: The range starts at or past the end of the file (`range_not_satisfiable`); `Content-Range` states the size.

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

Status `421`: Another node opened this session (`misdirected`).

```json
{
  "error": "misdirected"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

Status `503`: Storage is unavailable; retry (`unavailable`). When the sign-in could not be checked because the library did not answer in time, the answer is `session_check_unavailable` instead, with `Retry-After`; ask again rather than treating the caller as signed out.

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

## `GET /api/playback/progress`

Lists the caller's progress, most recently played first.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `limit` (query) *integer (int64)* — Rows per page: 1 to 100, default 20.
- `cursor` (query) *string* — `next_cursor` of the page before.
- `include_finished` (query) *boolean* — True also lists editions the caller finished; default false (Continue listening).

### Responses

- `200` — A page of the caller's progress. Body: [`ProgressPage`](schemas.md#progresspage) (`application/json`)
  - `items` *array of [`ProgressJson`](schemas.md#progressjson)*, **required** — The caller's progress rows, most recently played first. Editions their patron scope no longer shows are left out, so a page can be shorter than `limit` with more to come.
  - `next_cursor` *string or null* — Cursor of the next page; absent on the last.
- `400` — The cursor is not one this route returned. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be read. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/progress' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/progress' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/playback/progress",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/progress", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/playback/progress")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of the caller's progress.

```json
{
  "items": [
    {
      "edition_id": "5d2c8e1a-7b3f-4a6e-9c0d-1e2f3a4b5c6d",
      "work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b",
      "position_ms": 754000,
      "duration_ms": 41400000,
      "is_finished": false,
      "started_at": "2026-11-19T08:15:00+00:00",
      "finished_at": null,
      "updated_at": "2026-11-19T09:02:41+00:00"
    }
  ],
  "next_cursor": "MjAyNi0xMS0xOVQwOTowMjo0MSswMDowMHw1ZDJj"
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/playback/progress` answer.",
  "required": [
    "items"
  ],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One person's progress in one edition.",
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
      "description": "The caller's progress rows, most recently played first. Editions\ntheir patron scope no longer shows are left out, so a page can be\nshorter than `limit` with more to come."
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Cursor of the next page; absent on the last.",
      "example": "MjAyNi0xMS0xOVQwOTowMjo0MSswMDowMHw1ZDJj"
    }
  }
}
```

Status `400`: The cursor is not one this route returned.

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

## `PUT /api/playback/progress/{edition_id}`

Sets the caller's progress in an edition outright: Mark finished,
Start over, or a place chosen with no session open.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `edition_id` (path) *string*, **required** — Edition id.

### Request body

`application/json`: [`ProgressEditBody`](schemas.md#progresseditbody)

- `position_ms` *integer (int64) or null* — Position in milliseconds (at most sixty days). At least 0.
- `duration_ms` *integer (int64) or null* — The edition's runtime in milliseconds, when the page knows it. At least 0.
- `is_finished` *boolean or null* — True marks the edition and its work finished; false clears the edition's flag (the work's stays, and is changed with the work's own state). Left out, the position decides.

### Responses

- `200` — The caller's progress after the change. Body: [`SyncJson`](schemas.md#syncjson) (`application/json`)
  - `progress` *[`ProgressJson`](schemas.md#progressjson)*, **required** — The caller's progress after the report.
  - `finished_now` *boolean*, **required** — True when this report is the one that reached the edition's end: the edition and its work are now finished for the caller.
- `400` — A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The session ended. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such edition. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
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
$ curl 'http://127.0.0.1:8787/api/playback/progress/<edition_id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/progress/<edition_id>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/playback/progress/<edition_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/progress/<edition_id>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/playback/progress/<edition_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's progress after the change.

```json
{
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
  "finished_now": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/playback/sessions/{id}/sync` and `.../close` answer.",
  "required": [
    "progress",
    "finished_now"
  ],
  "properties": {
    "progress": {
      "type": "object",
      "description": "The caller's progress after the report.",
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
    "finished_now": {
      "type": "boolean",
      "description": "True when this report is the one that reached the edition's end:\nthe edition and its work are now finished for the caller.",
      "example": false
    }
  }
}
```

Status `400`: A length is past sixty days. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "position_ms must be at most 5184000000"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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
Failed to deserialize the JSON body into the target type: is_finished: invalid type: integer `1`, expected a boolean at line 1 column 16
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

## `DELETE /api/playback/progress/{edition_id}`

Forgets the caller's progress in an edition. The work's finished flag
is left as it is.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `edition_id` (path) *string*, **required** — Edition id.

### Responses

- `204` — The caller has no progress in the edition now. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `500` — The library could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/progress/<edition_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/progress/<edition_id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/playback/progress/<edition_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/progress/<edition_id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/playback/progress/<edition_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The caller has no progress in the edition now. No body.

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

Status `500`: The library could not be written.

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

## `PUT /api/playback/bookmarks/{id}`

Changes one of the caller's bookmarks.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Bookmark id.

### Request body

`application/json`: [`BookmarkEditBody`](schemas.md#bookmarkeditbody)

- `position_ms` *integer (int64) or null* — The new place, in milliseconds. At least 0.
- `title` *string or null* — The new note; an empty string clears it.

### Responses

- `200` — The bookmark after the change. Body: [`BookmarkJson`](schemas.md#bookmarkjson) (`application/json`)
  - `id` *string*, **required** — Bookmark id.
  - `edition_id` *string*, **required** — The edition it is in.
  - `position_ms` *integer (int64)*, **required** — The marked place, in milliseconds from the edition's start. At least 0.
  - `title` *string*, **required** — The caller's note; may be empty.
  - `created_at` *string*, **required** — When it was made (RFC 3339).
- `400` — The position is past sixty days, or the title is too long or has control characters. As `text/plain`, when the body is not syntactically valid JSON. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such bookmark of the caller's (`unknown_bookmark`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is over 4096 bytes. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/bookmarks/<id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/bookmarks/<id>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/playback/bookmarks/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/bookmarks/<id>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/playback/bookmarks/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The bookmark after the change.

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

Status `404`: No such bookmark of the caller's (`unknown_bookmark`).

```json
{
  "error": "unknown_bookmark"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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
Failed to deserialize the JSON body into the target type: title: invalid type: integer `1`, expected a string at line 1 column 10
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be written.

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

## `DELETE /api/playback/bookmarks/{id}`

Removes one of the caller's bookmarks.

**Access:** The operator, or any library session, including a library card (refused with 403 `transport_insecure` off loopback without TLS, or `clerk_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clerkToken`

### Parameters

- `id` (path) *string*, **required** — Bookmark id.

### Responses

- `204` — The bookmark is gone. No body.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The session has no library user. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `clerk_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorMessage`](schemas.md#errormessage) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) or [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `404` — No such bookmark of the caller's (`unknown_bookmark`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `500` — The library could not be written. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be checked because the library did not answer in time (`session_check_unavailable`). The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/playback/bookmarks/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/playback/bookmarks/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/playback/bookmarks/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/playback/bookmarks/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/playback/bookmarks/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: The bookmark is gone. No body.

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

Status `404`: No such bookmark of the caller's (`unknown_bookmark`).

```json
{
  "error": "unknown_bookmark"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `500`: The library could not be written.

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
