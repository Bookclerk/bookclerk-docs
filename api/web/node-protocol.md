# Node protocol

Node-to-node enrollment, socket, artifacts and uploads; each route checks its own credential.

Back to the [web API reference](README.md).

## `POST /api/node/v1/enroll`

`POST /api/node/v1/enroll`: redeem an enrollment link. Answers
`{cluster_id, node_id}`; 403 `transport_insecure` when the connection
fails the transport policy, 403 `enrollment_refused` for any link that
does not redeem.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`EnrollBody`](schemas.md#enrollbody)

- `link_id` *string*, **required** — Link id from the token.
- `secret` *string*, **required** — Link secret from the token.
- `sign_pk` *string*, **required** — Ed25519 public signing key.
- `box_pk` *string*, **required** — X25519 public box key.
- `software_version` *string* — Host's Bookclerk version.

### Responses

- `200` — The host is enrolled; its node credential. Body: [`EnrolledNodeView`](schemas.md#enrollednodeview) (`application/json`)
  - `cluster_id` *string*, **required** — The cluster the node joined.
  - `node_id` *string*, **required** — The node id minted for it.
- `400` — The body is not an enroll request, or its keys are not base64url. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the link does not redeem: unknown, used, expired, revoked, or enrollment is off (`enrollment_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`NodeRefusal`](schemas.md#noderefusal) (`application/json`)
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is larger than 16 KiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request is not `application/json`. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The enrollment could not be read or recorded; try again (`enrollment_unavailable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/enroll' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/enroll' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/node/v1/enroll",
    json={"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/enroll", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/node/v1/enroll")
    .json(&serde_json::json!({"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The host is enrolled; its node credential.

```json
{
  "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body after an enrollment link or branch sign-in was redeemed.",
  "required": [
    "cluster_id",
    "node_id"
  ],
  "properties": {
    "cluster_id": {
      "type": "string",
      "description": "The cluster the node joined.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    },
    "node_id": {
      "type": "string",
      "description": "The node id minted for it.",
      "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
    }
  }
}
```

Status `400`: The body is not an enroll request, or its keys are not base64url.

```json
{
  "error": "sign_pk and box_pk must be base64url"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the link does not redeem: unknown, used, expired, revoked, or enrollment is off (`enrollment_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "enrollment_refused"
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

Status `413`: The body is larger than 16 KiB.

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

Status `503`: The enrollment could not be read or recorded; try again (`enrollment_unavailable`).

```json
{
  "error": "enrollment_unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

## `GET /api/node/v1/info`

`GET /api/node/v1/info`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — What this library says about itself to a joining install. Body: [`OwnerInfo`](schemas.md#ownerinfo) (`application/json`)
  - `cluster_id` *string*, **required** — The library's cluster id.
  - `library_name` *string*, **required** — A name to show for the library.
  - `origin` *string*, **required** — The library's canonical origin: the one the node handshake signs and the authorize endpoint lives under.
  - `transport` *string*, **required** — The node API transport mode (`acme`, `proxy`, `self_signed`, `loopback`).
  - `enrollment` *boolean*, **required** — Whether the library lets new branches join (`core.host_enrollment`).
  - `spki_fingerprint` *string or null* — In `self_signed` mode: base64url SHA-256 of the certificate's SubjectPublicKeyInfo, which the joining operator confirms against the owner's Settings → Branches page.
  - `software_version` *string* — The library's Bookclerk version.
  - `ancestry` *array of string* — The library's ancestry: its own cluster id first, then the cluster id of every library above it up to the root. A joining library refuses a parent whose ancestry names its own cluster id, so a tree of libraries never becomes a cycle.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`). Body: [`TransportRefused`](schemas.md#transportrefused) (`application/json`)
  - `error` *string*, **required** — Always `transport_insecure`.
  - `reason` *string*, **required** — The transport check that refused the connection (`plain_http`, `unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`, `proxy_proto_ambiguous`, `proxy_proto_not_https`, `public_origin_not_https`, `listener_mode_mismatch`, `node_api_misconfigured`).
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — This library's cluster row (`cluster_unavailable`) or origin (`origin_unavailable`) is not available yet. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/info'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/info'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/node/v1/info")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/info");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/node/v1/info").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What this library says about itself to a joining install.

```json
{
  "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "library_name": "books.example.com",
  "origin": "https://books.example.com",
  "transport": "acme",
  "enrollment": true,
  "spki_fingerprint": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o",
  "software_version": "0.4.0",
  "ancestry": [
    "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "`GET /api/node/v1/info` answer.",
  "required": [
    "cluster_id",
    "library_name",
    "origin",
    "transport",
    "enrollment"
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
      "description": "The library's canonical origin: the one the node handshake signs and\nthe authorize endpoint lives under.",
      "example": "https://books.example.com"
    },
    "transport": {
      "type": "string",
      "description": "The node API transport mode (`acme`, `proxy`, `self_signed`,\n`loopback`).",
      "example": "acme"
    },
    "enrollment": {
      "type": "boolean",
      "description": "Whether the library lets new branches join (`core.host_enrollment`).",
      "example": true
    },
    "spki_fingerprint": {
      "type": [
        "string",
        "null"
      ],
      "description": "In `self_signed` mode: base64url SHA-256 of the certificate's\nSubjectPublicKeyInfo, which the joining operator confirms against\nthe owner's Settings → Branches page.",
      "example": "kQ3nUj9Z2vY4xW8pT6rS1aB5cD7eF0gH2iJ4kL6mN8o"
    },
    "software_version": {
      "type": "string",
      "description": "The library's Bookclerk version.",
      "example": "0.4.0"
    },
    "ancestry": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "The library's ancestry: its own cluster id first, then the cluster\nid of every library above it up to the root. A joining library\nrefuses a parent whose ancestry names its own cluster id, so a\ntree of libraries never becomes a cycle.",
      "example": [
        "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
      ]
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`).

```json
{
  "error": "transport_insecure",
  "reason": "plain_http"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a `403` from the node API's transport policy: the\nconnection is plain HTTP off loopback, or arrived through a proxy the\npolicy does not trust, and `reason` names the check that refused it.",
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

## `POST /api/node/v1/enroll/oauth`

`POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in.
Answers `{cluster_id, node_id}` for a branch, plus `connection` and the
sealed `bootstrap` for a clerk; 403 `transport_insecure` for a refused
connection, 403 `enrollment_refused` for a code that does not redeem
(unknown, used, expired, another client's, a wrong verifier or
redirect URI, enrollment off, or the user no longer active).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`OAuthEnrollRequest`](schemas.md#oauthenrollrequest)

- `code` *string*, **required** — The authorization code from the callback.
- `code_verifier` *string*, **required** — The PKCE verifier.
- `redirect_uri` *string*, **required** — The redirect URI the authorization was requested with.
- `sign_pk` *string*, **required** — Ed25519 public signing key.
- `box_pk` *string*, **required** — X25519 public box key.
- `branch_name` *string* — The name this branch joins as (the node's label).
- `software_version` *string* — The joining install's Bookclerk version.
- `connection` *[`ConnectionType`](schemas.md#connectiontype)* — The connection type the code was approved for. It must match the one bound to the code at consent; left out for a branch.

### Responses

- `200` — The install is enrolled: its cluster and node ids for a branch, and for a clerk also `connection` and the clerk bootstrap sealed to its box key. Body: [`OAuthEnrollAnswer`](schemas.md#oauthenrollanswer) (`application/json`)
  - `cluster_id` *string*, **required** — The library's cluster id.
  - `node_id` *string*, **required** — The node id recorded for the joining install.
  - `connection` *[`ConnectionType`](schemas.md#connectiontype)* — The connection type recorded (`branch` when left out).
  - `bootstrap` *string or null* — For a clerk: the [`crate::ClerkBootstrap`] sealed to the node's box key ([`crate::seal_clerk_bootstrap`]), as base64url without padding.
- `400` — The body is not an enroll request, or its keys are not base64url. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the code does not redeem: blank, unknown, used, expired, wrong verifier, or the user may no longer let a clerk join (`enrollment_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`NodeRefusal`](schemas.md#noderefusal) (`application/json`)
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — This library's database cannot be shared with a clerk (`database_not_shareable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `413` — The body is larger than 16 KiB. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `415` — The request is not `application/json`. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — The enrollment could not be read or recorded; try again (`enrollment_unavailable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/enroll/oauth' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/enroll/oauth' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/node/v1/enroll/oauth",
    json={"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/enroll/oauth", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/node/v1/enroll/oauth")
    .json(&serde_json::json!({"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The install is enrolled: its cluster and node ids for a branch, and for a clerk also `connection` and the clerk bootstrap sealed to its box key.

```json
{
  "cluster_id": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f",
  "node_id": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/node/v1/enroll/oauth` answer.",
  "required": [
    "cluster_id",
    "node_id"
  ],
  "properties": {
    "cluster_id": {
      "type": "string",
      "description": "The library's cluster id.",
      "example": "c1f2e3d4-5a6b-4c7d-8e9f-0a1b2c3d4e5f"
    },
    "node_id": {
      "type": "string",
      "description": "The node id recorded for the joining install.",
      "example": "7d2b9e4c-1f3a-4b8d-9c6e-2a5f8b1d4e7c"
    },
    "connection": {
      "type": "string",
      "description": "The connection type recorded (`branch` when left out).",
      "enum": [
        "branch",
        "clerk"
      ]
    },
    "bootstrap": {
      "type": [
        "string",
        "null"
      ],
      "description": "For a clerk: the [`crate::ClerkBootstrap`] sealed to the node's box\nkey ([`crate::seal_clerk_bootstrap`]), as base64url without padding.",
      "example": "Qm9va2NsZXJrIHNlYWxlZCBjbGVyayBib290c3RyYXA6IGVwaGVtZXJhbCBrZXkgKyBjaXBoZXJ0ZXh0"
    }
  }
}
```

Status `400`: The body is not an enroll request, or its keys are not base64url.

```json
{
  "error": "sign_pk and box_pk must be base64url"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`), or the code does not redeem: blank, unknown, used, expired, wrong verifier, or the user may no longer let a clerk join (`enrollment_refused`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "enrollment_refused"
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

Status `409`: This library's database cannot be shared with a clerk (`database_not_shareable`).

```json
{
  "error": "database_not_shareable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `413`: The body is larger than 16 KiB.

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

Status `503`: The enrollment could not be read or recorded; try again (`enrollment_unavailable`).

```json
{
  "error": "enrollment_unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
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

## `GET /api/node/v1/connect`

`GET /api/node/v1/connect`: upgrade to the host-node socket after the
transport check. 403 `transport_insecure` for a refused connection,
400 `subprotocol_required` without [`SUBPROTOCOL`].

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `101` — Switching to the host-node WebSocket. The node's signature is checked in the first frames; a wrong one closes the socket. No body.
- `400` — The request does not offer the node subprotocol (`subprotocol_required`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The transport is not allowed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — Too many handshakes are in flight (`handshakes_busy`); retry after `Retry-After` seconds. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/connect'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/connect'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/node/v1/connect")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/connect");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/node/v1/connect").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `101`: Switching to the host-node WebSocket. The node's signature is checked in the first frames; a wrong one closes the socket. No body.

Status `400`: The request does not offer the node subprotocol (`subprotocol_required`).

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

Status `403`: The transport is not allowed.

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

Status `404`: The node API is not mounted.

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

Status `503`: Too many handshakes are in flight (`handshakes_busy`); retry after `Retry-After` seconds.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/node/v1/artifacts/{sha256}`

`GET /api/node/v1/artifacts/{sha256}`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `nodeSignature`

### Parameters

- `sha256` (path) *string*, **required** — Archive digest, lowercase hex.

### Responses

- `200` — The plugin archive. Body: [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `401` — The node signature is missing or wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such archive, or the node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>' \
    -H 'Authorization: Bookclerk-Node <node_id> <issued_at> <signature>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>' `
    -Headers @{ Authorization = 'Bookclerk-Node <node_id> <issued_at> <signature>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>",
    headers={"Authorization": "Bookclerk-Node <node_id> <issued_at> <signature>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>", {
  headers: { Authorization: "Bookclerk-Node <node_id> <issued_at> <signature>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>")
    .header("Authorization", "Bookclerk-Node <node_id> <issued_at> <signature>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The plugin archive. The body is binary (`application/octet-stream`).

Schema:

A `application/octet-stream` body: binary bytes.

Status `401`: The node signature is missing or wrong.

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

Status `404`: No such archive, or the node API is not mounted.

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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `PUT /api/node/v1/uploads/{upload_id}/files/{index}`

`PUT /api/node/v1/uploads/{upload_id}/files/{index}`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `uploadToken`

### Parameters

- `upload_id` (path) *string*, **required** — Upload id.
- `index` (path) *integer (int32)*, **required** — File index in the upload.

### Request body

`application/octet-stream`: [`BinaryBody`](schemas.md#binarybody)

### Responses

- `200` — The file was stored (`received`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `401` — The upload token is missing or wrong, or names no live upload (`unauthorized`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`). Body: [`TransportRefused`](schemas.md#transportrefused) (`application/json`)
  - `error` *string*, **required** — Always `transport_insecure`.
  - `reason` *string*, **required** — The transport check that refused the connection (`plain_http`, `unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`, `proxy_proto_ambiguous`, `proxy_proto_not_https`, `public_origin_not_https`, `listener_mode_mismatch`, `node_api_misconfigured`).
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The upload lost its fence to a newer one (`lost`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `421` — This node is not the upload's receiving node (`misdirected`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `422` — The file was refused (`refused`): not in the upload (`unknown_file`), a size or content mismatch (`size_mismatch`, `content_mismatch`), the destinations changed (`destinations_changed`), or a granted storage key now belongs to another title or writer (`key_taken`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `503` — Storage or the upload record is unavailable, or the file lock timed out; retry (`unavailable`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>' \
    -X PUT \
    -H 'Authorization: Bookclerk-Upload <token>' \
    -H 'Content-Type: application/octet-stream' \
    --data-binary @file
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>' `
    -Method Put `
    -Headers @{ Authorization = 'Bookclerk-Upload <token>' } `
    -ContentType 'application/octet-stream' `
    -InFile 'file'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>",
    headers={"Authorization": "Bookclerk-Upload <token>", "Content-Type": "application/octet-stream"},
    data=open("file", "rb"),
)
print(response.status_code, response.text)
```

```javascript
import { readFile } from "node:fs/promises";

const response = await fetch("http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>", {
  method: "PUT",
  headers: { Authorization: "Bookclerk-Upload <token>", "Content-Type": "application/octet-stream" },
  body: await readFile("file"),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>")
    .header("Authorization", "Bookclerk-Upload <token>")
    .header("Content-Type", "application/octet-stream")
    .body(std::fs::read("file")?)
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The file was stored (`received`).

```json
{
  "job_completed": false,
  "status": "received"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `401`: The upload token is missing or wrong, or names no live upload (`unauthorized`).

```json
{
  "job_completed": false,
  "status": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`).

```json
{
  "error": "transport_insecure",
  "reason": "plain_http"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a `403` from the node API's transport policy: the\nconnection is plain HTTP off loopback, or arrived through a proxy the\npolicy does not trust, and `reason` names the check that refused it.",
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

Status `409`: The upload lost its fence to a newer one (`lost`).

```json
{
  "job_completed": false,
  "status": "lost"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `421`: This node is not the upload's receiving node (`misdirected`).

```json
{
  "job_completed": false,
  "status": "misdirected"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `422`: The file was refused (`refused`): not in the upload (`unknown_file`), a size or content mismatch (`size_mismatch`, `content_mismatch`), the destinations changed (`destinations_changed`), or a granted storage key now belongs to another title or writer (`key_taken`).

```json
{
  "job_completed": false,
  "reason": "content_mismatch",
  "status": "refused"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `503`: Storage or the upload record is unavailable, or the file lock timed out; retry (`unavailable`).

```json
{
  "job_completed": false,
  "status": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

## `POST /api/node/v1/uploads/{upload_id}/commit`

`POST /api/node/v1/uploads/{upload_id}/commit`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `uploadToken`

### Parameters

- `upload_id` (path) *string*, **required** — Upload id.

### Request body

`application/json`: [`UploadCommitRequest`](schemas.md#uploadcommitrequest)

- `finish` *boolean* — Complete the job with this commit (the job's closing upload).
- `detail` *string or null* — Outcome text recorded as the job's progress when `finish` is set.

### Responses

- `200` — The upload was committed to the destinations (`committed`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `401` — The upload token is missing or wrong, or names no live upload (`unauthorized`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`). Body: [`TransportRefused`](schemas.md#transportrefused) (`application/json`)
  - `error` *string*, **required** — Always `transport_insecure`.
  - `reason` *string*, **required** — The transport check that refused the connection (`plain_http`, `unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`, `proxy_proto_ambiguous`, `proxy_proto_not_https`, `public_origin_not_https`, `listener_mode_mismatch`, `node_api_misconfigured`).
- `404` — The node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `409` — The upload lost its fence to a newer one (`lost`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `421` — This node is not the upload's receiving node (`misdirected`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `422` — The commit was refused (`refused`), with the cause in `reason`: the body does not parse (`invalid_body`); the upload is unknown, aborted, or its job was cancelled (`unknown_upload`, `upload_aborted`, `job_cancelled`); not every granted file arrived (`files_missing`); the destinations changed since the grant (`destinations_changed`); a storage key now belongs to another title or writer, or the title or book is gone (`key_taken`, `unknown_title`, `unknown_book`); or the closing upload is mismatched: an ordinary upload claims to finish the job (`not_a_closing_upload`), the closing upload does not (`closing_upload_must_finish`), or it arrives while titles are still pending (`titles_remaining`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `503` — Storage or the upload record is unavailable; retry (`unavailable`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit' \
    -X POST \
    -H 'Authorization: Bookclerk-Upload <token>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit' `
    -Method Post `
    -Headers @{ Authorization = 'Bookclerk-Upload <token>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit",
    headers={"Authorization": "Bookclerk-Upload <token>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit", {
  method: "POST",
  headers: { Authorization: "Bookclerk-Upload <token>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit")
    .header("Authorization", "Bookclerk-Upload <token>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The upload was committed to the destinations (`committed`).

```json
{
  "job_completed": true,
  "status": "committed"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `401`: The upload token is missing or wrong, or names no live upload (`unauthorized`).

```json
{
  "job_completed": false,
  "status": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`).

```json
{
  "error": "transport_insecure",
  "reason": "plain_http"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a `403` from the node API's transport policy: the\nconnection is plain HTTP off loopback, or arrived through a proxy the\npolicy does not trust, and `reason` names the check that refused it.",
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

Status `409`: The upload lost its fence to a newer one (`lost`).

```json
{
  "job_completed": false,
  "status": "lost"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `421`: This node is not the upload's receiving node (`misdirected`).

```json
{
  "job_completed": false,
  "status": "misdirected"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `422`: The commit was refused (`refused`), with the cause in `reason`: the body does not parse (`invalid_body`); the upload is unknown, aborted, or its job was cancelled (`unknown_upload`, `upload_aborted`, `job_cancelled`); not every granted file arrived (`files_missing`); the destinations changed since the grant (`destinations_changed`); a storage key now belongs to another title or writer, or the title or book is gone (`key_taken`, `unknown_title`, `unknown_book`); or the closing upload is mismatched: an ordinary upload claims to finish the job (`not_a_closing_upload`), the closing upload does not (`closing_upload_must_finish`), or it arrives while titles are still pending (`titles_remaining`).

```json
{
  "job_completed": false,
  "reason": "closing_upload_must_finish",
  "status": "refused"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
    }
  }
}
```

Status `503`: Storage or the upload record is unavailable; retry (`unavailable`).

```json
{
  "job_completed": false,
  "status": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Answer of the upload endpoint (every status carries one).",
  "required": [
    "status"
  ],
  "properties": {
    "status": {
      "type": "string",
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`.",
      "example": "received"
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`.",
      "example": "size_mismatch"
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished.",
      "example": false
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
