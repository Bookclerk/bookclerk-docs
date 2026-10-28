# Library

Books in the library, scans and acquires.

Back to the [web API reference](README.md).

## `POST /api/library/scan`

Plans and enqueues a library scan; returns the accepted job ids.

The scan planner splits a scan of every account into one job per
scan-enabled account for operators, Owners and Administrators, and
limits anyone else to the accounts they own. `job_id` is the first
accepted (or already queued) job and `job_ids` lists every one.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`ScanRequest`](schemas.md#scanrequest) or null

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `account` | string or null | no | When set, scan this account even if `scan_enabled` is false. |
| `plugin_instance_id` | string or null | no | When set, scan only this deployed plugin instance. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The scan jobs were accepted. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 400 | The body names no valid target. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not scan these accounts. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 409 | A scan is already pending or running. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 422 | No account matched. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 429 | The job queue is full. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/scan' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/scan",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/scan", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/scan")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The scan jobs were accepted.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "job_ids": [
    "string"
  ],
  "error": "string"
}
```

Status `400`: The body names no valid target.

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

Status `403`: The caller may not scan these accounts. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `409`: A scan is already pending or running.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "job_ids": [
    "string"
  ],
  "error": "string"
}
```

Status `422`: No account matched.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "job_ids": [
    "string"
  ],
  "error": "string"
}
```

Status `429`: The job queue is full.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "job_ids": [
    "string"
  ],
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/library/acquire`

Plans and enqueues acquire jobs; returns the accepted job ids.

The acquire planner makes one job per title id, and limits a user who is
not an Owner or Administrator to the accounts they own (one job per
account when no title or account is named). `job_id` is the first
accepted (or already queued) job and `job_ids` lists every one.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`AcquireRequestBody`](schemas.md#acquirerequestbody) or null

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string or null | no | Audible ASIN filter; first of uuid/asin/isbn/product_id wins. |
| `uuid` | string or null | no | Library title UUID filter. |
| `isbn` | string or null | no | ISBN filter for non-Audible stores. |
| `product_id` | string or null | no | Store product id filter. |
| `account` | string or null | no | Restrict acquire to this store account. |
| `plugin_instance_id` | string or null | no | When set, fetch through this deployed plugin instance. |
| `titles` | array of string | no | More title ids; each (with the single filter above) becomes its own job. |
| `force` | boolean | no | Re-download titles already acquired. |
| `pdf_only` | boolean | no | Fetch only each title's companion PDF. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The acquire jobs were accepted. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 400 | The body names no valid title. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not acquire for these accounts. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 409 | The same acquire is already pending or running, or no node can run it. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 429 | The job queue is full. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 500 | The jobs could not be queued. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 503 | Job admission is paused. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/acquire' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/acquire' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/library/acquire",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/acquire", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/library/acquire")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The acquire jobs were accepted.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `400`: The body names no valid title.

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

Status `403`: The caller may not acquire for these accounts. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `409`: The same acquire is already pending or running, or no node can run it.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `429`: The job queue is full.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `500`: The jobs could not be queued.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `503`: Job admission is paused.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/library/books`

Lists or searches library titles with optional account/status filters and pagination.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `account` | query | string | no | Restrict results to this store account id. |
| `status` | query | string | no | Optional `AcquireStatus` wire-string filter. |
| `q` | query | string | no | Full-text search query; empty skips the search index. |
| `limit` | query | integer | no | Page size, clamped to 1–500 (default 40). |
| `offset` | query | integer | no | Number of matching titles to skip. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A page of titles. | [`BooksResponse`](schemas.md#booksresponse) (`application/json`) |
| 400 | A filter is invalid. | any JSON (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The library could not be read. | any JSON (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of titles.

```json
{
  "books": [
    {
      "id": 0,
      "uuid": "string",
      "plugin_key": "string",
      "source": "string",
      "account_id": "string",
      "product_id": "string",
      "asin": "string",
      "isbn": "string",
      "marketplace": "string",
      "title": "string",
      "authors": "string",
      "narrators": "string",
      "series": "string",
      "series_index": "string",
      "series_asin": "string",
      "acquire_status": "not_acquired",
      "storage_key": "string",
      "error_message": "string",
      "purchased_at": "2026-01-01T12:00:00Z",
      "tags": "string",
      "rating_overall": 0,
      "rating_performance": 0,
      "rating_story": 0,
      "is_finished": true,
      "pdf_status": "not_acquired",
      "pdf_storage_key": "string",
      "publisher": "string",
      "length_minutes": 0,
      "is_abridged": true,
      "content_kind": "string",
      "categories": "string",
      "subtitle": "string",
      "published_at": "2026-01-01T12:00:00Z",
      "description": "string",
      "language": "string",
      "cover_url": "string",
      "subjects": "string",
      "enrich_source": "string",
      "enrich_confidence": 0,
      "enrich_updated_at": "2026-01-01T12:00:00Z",
      "created_at": "2026-01-01T12:00:00Z",
      "updated_at": "2026-01-01T12:00:00Z"
    }
  ],
  "total": 0,
  "limit": 0,
  "offset": 0
}
```

Status `400`: A filter is invalid. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The library could not be read. The body is JSON the schemas do not describe yet.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/library/books/{uuid}`

Returns one title by UUID, or 404 when it is absent.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `uuid` | path | string | yes | Title UUID. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The title. | [`BookRecord`](schemas.md#bookrecord) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such title. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The library could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books/<uuid>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books/<uuid>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books/<uuid>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books/<uuid>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books/<uuid>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The title.

```json
{
  "id": 0,
  "uuid": "string",
  "plugin_key": "string",
  "source": "string",
  "account_id": "string",
  "product_id": "string",
  "asin": "string",
  "isbn": "string",
  "marketplace": "string",
  "title": "string",
  "authors": "string",
  "narrators": "string",
  "series": "string",
  "series_index": "string",
  "series_asin": "string",
  "acquire_status": "not_acquired",
  "storage_key": "string",
  "error_message": "string",
  "purchased_at": "2026-01-01T12:00:00Z",
  "tags": "string",
  "rating_overall": 0,
  "rating_performance": 0,
  "rating_story": 0,
  "is_finished": true,
  "pdf_status": "not_acquired",
  "pdf_storage_key": "string",
  "publisher": "string",
  "length_minutes": 0,
  "is_abridged": true,
  "content_kind": "string",
  "categories": "string",
  "subtitle": "string",
  "published_at": "2026-01-01T12:00:00Z",
  "description": "string",
  "language": "string",
  "cover_url": "string",
  "subjects": "string",
  "enrich_source": "string",
  "enrich_confidence": 0,
  "enrich_updated_at": "2026-01-01T12:00:00Z",
  "created_at": "2026-01-01T12:00:00Z",
  "updated_at": "2026-01-01T12:00:00Z"
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

Status `404`: No such title.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The library could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/library/books/{uuid}/cover`

Serves the JPEG cover sidecar from the deployed local destination, or 404
when no local destination runs on this node.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `uuid` | path | string | yes | Title UUID. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The cover. | binary (`image/jpeg`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such title, or it has no cover. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/library/books/<uuid>/cover' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/library/books/<uuid>/cover' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/library/books/<uuid>/cover",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/library/books/<uuid>/cover", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/library/books/<uuid>/cover")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The cover. The body is `image/jpeg`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `404`: No such title, or it has no cover.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
