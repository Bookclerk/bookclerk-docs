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

- `200` — The host is enrolled; its node credential. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The link is unknown, used or expired. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The host is enrolled; its node credential. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `401`: The link is unknown, used or expired.

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

Status `403`: The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: The node API is not mounted.

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

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/node/v1/info`

`GET /api/node/v1/info`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — What this library says about itself to a joining install. Body: any JSON (`application/json`)
- `403` — The transport is not allowed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: What this library says about itself to a joining install. The body is JSON the schemas do not describe yet.

Status `403`: The transport is not allowed.

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

Status `404`: The node API is not mounted.

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

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/node/v1/enroll/oauth`

`POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in.
Answers `{cluster_id, node_id}`; 403 `transport_insecure` for a refused
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

- `200` — The branch is enrolled; its node credential. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The code or verifier is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The enrollment could not be recorded; try again (`enrollment_unavailable`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The branch is enrolled; its node credential. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `401`: The code or verifier is wrong.

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

Status `403`: The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: The node API is not mounted.

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

Status `503`: The enrollment could not be recorded; try again (`enrollment_unavailable`).

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

Status `504`: The request ran past the API's time budget.

```text
string
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

Status `403`: The transport is not allowed.

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

Status `404`: The node API is not mounted.

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

Status `503`: Too many handshakes are in flight (`handshakes_busy`); retry after `Retry-After` seconds.

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

Status `504`: The request ran past the API's time budget.

```text
string
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

Status `404`: No such archive, or the node API is not mounted.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `401` — The upload token is missing or wrong (`unauthorized`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `403` — The connection's transport is refused (`transport_insecure`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such upload, or the node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `401`: The upload token is missing or wrong (`unauthorized`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `403`: The connection's transport is refused (`transport_insecure`).

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

Status `404`: No such upload, or the node API is not mounted.

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

Status `409`: The upload lost its fence to a newer one (`lost`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `421`: This node is not the upload's receiving node (`misdirected`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `422`: The file was refused (`refused`): not in the upload (`unknown_file`), a size or content mismatch (`size_mismatch`, `content_mismatch`), the destinations changed (`destinations_changed`), or a granted storage key now belongs to another title or writer (`key_taken`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `503`: Storage or the upload record is unavailable, or the file lock timed out; retry (`unavailable`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
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
- `401` — The upload token is missing or wrong (`unauthorized`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `403` — The connection's transport is refused (`transport_insecure`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such upload, or the node API is not mounted. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The upload lost its fence to a newer one (`lost`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `421` — This node is not the upload's receiving node (`misdirected`). Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
  - `status` *string*, **required** — Stable outcome: `received`, `committed`, `lost`, `refused`, `misdirected`, `unauthorized`, `unavailable`.
  - `reason` *string or null* — Why, for `refused`.
  - `job_completed` *boolean* — For `committed`: the job is finished.
- `422` — The commit was refused (`refused`): an invalid body, missing files, or the destinations changed. Body: [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`)
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
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `401`: The upload token is missing or wrong (`unauthorized`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `403`: The connection's transport is refused (`transport_insecure`).

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

Status `404`: No such upload, or the node API is not mounted.

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

Status `409`: The upload lost its fence to a newer one (`lost`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `421`: This node is not the upload's receiving node (`misdirected`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `422`: The commit was refused (`refused`): an invalid body, missing files, or the destinations changed.

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
    }
  }
}
```

Status `503`: Storage or the upload record is unavailable; retry (`unavailable`).

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
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
      "description": "Stable outcome: `received`, `committed`, `lost`, `refused`,\n`misdirected`, `unauthorized`, `unavailable`."
    },
    "reason": {
      "type": [
        "string",
        "null"
      ],
      "description": "Why, for `refused`."
    },
    "job_completed": {
      "type": "boolean",
      "description": "For `committed`: the job is finished."
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
