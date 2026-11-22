# Webhooks

Inbound webhooks for plugins, outside operator auth: the URL token is the credential.

Back to the [web API reference](README.md).

## `POST /hooks/{token}/{name}`

Stores one inbound webhook request for a plugin instance.

The token is the credential; a row with a signing rule also needs a
valid signature. The reply body never says more than the status.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Parameters

- `token` (path) *string*, **required** — The webhook's URL token: 43 characters of base64url, from the plugin's `WEBHOOKS.url`.
- `name` (path) *string*, **required** — The `[[webhooks]]` row's name.

### Request body

The request body, kept as sent, at most the row's `max_body_bytes`.

`*/*`: [`BinaryBody`](schemas.md#binarybody)

### Responses

- `204` — Stored; the plugin reads it on its next pull. No body.
- `400` — A kept header repeats, is too long or holds a control character. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The signature is missing or does not match. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The token or name matches no webhook. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `413` — The body is larger than the row allows. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `429` — The webhook is over its `rate_per_minute`. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The webhook cannot take requests now: webhooks are off, the signing secret is not set, this node does not run the instance, the library is unavailable, or the instance already holds its cap of waiting requests. Retry later. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.

### Example request

```console
$ curl 'http://127.0.0.1:8787/hooks/<token>/<name>' \
    -X POST \
    -H 'Content-Type: */*' \
    --data-binary @file
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/hooks/<token>/<name>' `
    -Method Post `
    -ContentType '*/*' `
    -InFile 'file'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/hooks/<token>/<name>",
    headers={"Content-Type": "*/*"},
    data=open("file", "rb"),
)
print(response.status_code, response.text)
```

```javascript
import { readFile } from "node:fs/promises";

const response = await fetch("http://127.0.0.1:8787/hooks/<token>/<name>", {
  method: "POST",
  headers: { "Content-Type": "*/*" },
  body: await readFile("file"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/hooks/<token>/<name>")
    .header("Content-Type", "*/*")
    .body(std::fs::read("file")?)
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `204`: Stored; the plugin reads it on its next pull. No body.

Status `400`: A kept header repeats, is too long or holds a control character.

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

Status `401`: The signature is missing or does not match.

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

Status `404`: The token or name matches no webhook.

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

Status `413`: The body is larger than the row allows.

```json
{
  "error": "payload_too_large",
  "message": "Payload Too Large",
  "status": 413
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

Status `429`: The webhook is over its `rate_per_minute`.

```json
{
  "error": "too_many_requests",
  "message": "Too many requests. Try again shortly.",
  "status": 429
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

Status `503`: The webhook cannot take requests now: webhooks are off, the signing secret is not set, this node does not run the instance, the library is unavailable, or the instance already holds its cap of waiting requests. Retry later.

```json
{
  "error": "service_unavailable",
  "message": "The Bookclerk daemon is not ready to handle this request.",
  "status": 503
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
