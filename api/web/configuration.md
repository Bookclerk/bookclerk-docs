# Configuration

Live configuration domains and reloads.

Back to the [web API reference](README.md).

## `POST /api/config/reload`

Reloads `config.toml` and republishes daemon runtime without a process restart.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The configuration was reloaded. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The reload failed; the old configuration stays live. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/reload' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/reload' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/reload",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/reload", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/reload")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The configuration was reloaded.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The reload failed; the old configuration stays live.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/config/domains/core.events`

`GET /api/config/domains/core.events`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The `core.events` settings. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.events' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.events' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/domains/core.events",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.events", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/domains/core.events")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The `core.events` settings. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `403`: The caller's role does not allow this route.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/domains/core.events`

`PUT /api/config/domains/core.events`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PutEventsDomainRequest`](schemas.md#puteventsdomainrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller read. A mismatch is a conflict. |
| `retention_days` | integer (int64) | yes | Replacement `[events]` document. At least 0. |
| `dead_letter_retention_days` | integer (int64) | yes | Replacement dead-letter retention. At least 0. |
| `concurrency` | integer (int32) | yes | Replacement concurrency cap. At least 0. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.events' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 0, "retention_days": 0, "dead_letter_retention_days": 0, "concurrency": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.events' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 0, "retention_days": 0, "dead_letter_retention_days": 0, "concurrency": 0}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/domains/core.events",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 0, "retention_days": 0, "dead_letter_retention_days": 0, "concurrency": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.events", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 0, "retention_days": 0, "dead_letter_retention_days": 0, "concurrency": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/domains/core.events")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 0, "retention_days": 0, "dead_letter_retention_days": 0, "concurrency": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/config/domains/core.output`

`GET /api/config/domains/core.output`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The `core.output` settings. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.output' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.output' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/domains/core.output",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.output", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/domains/core.output")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The `core.output` settings. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `403`: The caller's role does not allow this route.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/domains/core.output`

`PUT /api/config/domains/core.output`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PutOutputDomainRequest`](schemas.md#putoutputdomainrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller read. A mismatch is a conflict. |
| `document` | object | yes | Complete replacement document (schema version 1). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/domains/core.output' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 0, "document": {}}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/domains/core.output' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 0, "document": {}}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/domains/core.output",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 0, "document": {}},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/domains/core.output", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 0, "document": {}}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/domains/core.output")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 0, "document": {}}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
