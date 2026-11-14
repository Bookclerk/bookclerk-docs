# Client sign-in

A clerk's sign-in as a person: redeem an approved code for a session token.

Back to the [web API reference](README.md).

## `POST /api/client/v1/token`

`POST /api/client/v1/token`: redeem an approved client sign-in for a
session token. The code is spent by any attempt, matching or not.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`ClientTokenRequest`](schemas.md#clienttokenrequest)

- `code` *string*, **required** — The single-use code from the redirect.
- `code_verifier` *string*, **required** — The PKCE verifier whose S256 challenge the authorize request sent.
- `redirect_uri` *string*, **required** — The redirect URI the authorize request named.

### Responses

- `200` — The session token (shown once), this library's cluster id, the signed-in user and how long the session stays signed in unused. Body: [`ClientTokenAnswer`](schemas.md#clienttokenanswer) (`application/json`)
  - `token` *string*, **required** — The session token, sent as `Authorization: Bookclerk-Client <token>`. Shown once; the library keeps only its hash.
  - `cluster_id` *string*, **required** — The library's cluster id.
  - `user` *[`ClientUser`](schemas.md#clientuser)*, **required** — The signed-in person on that library.
  - `idle_secs` *integer (int64)*, **required** — Seconds the session stays signed in unused; each use resets it.
- `400` — The body is not a token request (`invalid_request`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the code does not redeem: blank, unknown, used, expired, another client's, a wrong verifier or redirect URI, or the user is no longer active (`sign_in_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`NodeRefusal`](schemas.md#noderefusal) (`application/json`)
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is larger than 8 KiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request is not `application/json`. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The sign-in could not be read or recorded; try again (`unavailable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/client/v1/token' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"code": "q8Zl0m3VnS2xR7bW4cYd1eTf6gUh9iJk5LoNpQrStUv", "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk", "redirect_uri": "http://127.0.0.1:41923/client/callback"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/client/v1/token' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"code": "q8Zl0m3VnS2xR7bW4cYd1eTf6gUh9iJk5LoNpQrStUv", "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk", "redirect_uri": "http://127.0.0.1:41923/client/callback"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/client/v1/token",
    json={"code": "q8Zl0m3VnS2xR7bW4cYd1eTf6gUh9iJk5LoNpQrStUv", "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk", "redirect_uri": "http://127.0.0.1:41923/client/callback"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/client/v1/token", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"code": "q8Zl0m3VnS2xR7bW4cYd1eTf6gUh9iJk5LoNpQrStUv", "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk", "redirect_uri": "http://127.0.0.1:41923/client/callback"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/client/v1/token")
    .json(&serde_json::json!({"code": "q8Zl0m3VnS2xR7bW4cYd1eTf6gUh9iJk5LoNpQrStUv", "code_verifier": "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk", "redirect_uri": "http://127.0.0.1:41923/client/callback"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The session token (shown once), this library's cluster id, the signed-in user and how long the session stays signed in unused.

```json
{
  "token": "Zm9vYmFyYmF6cXV4cXV1eGNvcmdlZ3JhdWx0Z2FycGx5",
  "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "user": {
    "id": 7,
    "display_name": "Riley"
  },
  "idle_secs": 7776000
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/client/v1/token` answer.",
  "required": [
    "token",
    "cluster_id",
    "user",
    "idle_secs"
  ],
  "properties": {
    "token": {
      "type": "string",
      "description": "The session token, sent as `Authorization: Bookclerk-Client <token>`.\nShown once; the library keeps only its hash.",
      "example": "Zm9vYmFyYmF6cXV4cXV1eGNvcmdlZ3JhdWx0Z2FycGx5"
    },
    "cluster_id": {
      "type": "string",
      "description": "The library's cluster id.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    },
    "user": {
      "type": "object",
      "description": "The signed-in person on that library.",
      "required": [
        "id"
      ],
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64",
          "description": "The user's id on the library.",
          "example": 7
        },
        "display_name": {
          "type": [
            "string",
            "null"
          ],
          "description": "The user's display name on the library, when set.",
          "example": "Riley"
        }
      }
    },
    "idle_secs": {
      "type": "integer",
      "format": "int64",
      "description": "Seconds the session stays signed in unused; each use resets it.",
      "example": 7776000
    }
  }
}
```

Status `400`: The body is not a token request (`invalid_request`).

```json
{
  "error": "invalid_request"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the code does not redeem: blank, unknown, used, expired, another client's, a wrong verifier or redirect URI, or the user is no longer active (`sign_in_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "sign_in_refused"
}
```

Schema:

```json
{
  "anyOf": [
    {
      "type": "object",
      "description": "The connection failed the transport policy.",
      "required": [
        "error",
        "reason"
      ],
      "properties": {
        "error": {
          "type": "string",
          "description": "Always `transport_insecure`.",
          "example": "transport_insecure"
        },
        "reason": {
          "type": "string",
          "description": "The transport check that refused the connection (`plain_http`,\n`unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`,\n`proxy_proto_ambiguous`, `proxy_proto_not_https`,\n`public_origin_not_https`, `listener_mode_mismatch`,\n`node_api_misconfigured`).",
          "example": "plain_http"
        }
      }
    },
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
      "description": "The handler refused with a slug in `error`.",
      "required": [
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
      "description": "The handler refused with a slug in `error` and a sentence in `detail`.",
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
  "description": "A `403` from a node API route: the transport policy's refusal, the standard body, or the handler's own error; the route's examples name each."
}
```

Status `404`: The node API is not mounted on this listener (`node_api_unmounted`).

```json
{
  "error": "node_api_unmounted"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `413`: The body is larger than 8 KiB.

```json
{
  "error": "request too large"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `415`: The request is not `application/json`.

```json
{
  "error": "expected application/json"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `503`: The sign-in could not be read or recorded; try again (`unavailable`).

```json
{
  "error": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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
