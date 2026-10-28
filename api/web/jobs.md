# Jobs

Queued and running control-plane jobs.

Back to the [web API reference](README.md).

## `GET /api/jobs`

Returns durable scan/acquire/listen_sync jobs, newest first.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Jobs, newest first. | array of [`JobInfo`](schemas.md#jobinfo) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The job table could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/jobs' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/jobs' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/jobs",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/jobs", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/jobs")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Jobs, newest first.

```json
[
  {
    "id": "string",
    "kind": "scan",
    "status": "pending",
    "detail": "string",
    "progress": "string",
    "attempt_count": 0,
    "max_attempts": 0,
    "error_kind": "string",
    "created_at": "string",
    "finished_at": "string"
  }
]
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

Status `500`: The job table could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/jobs/{id}/cancel`

Cancel a pending job or request cooperative stop of a running job.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Job id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The job was cancelled or asked to stop. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such job. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The job could not be updated. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/jobs/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/jobs/<id>/cancel' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/jobs/<id>/cancel",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/jobs/<id>/cancel", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/jobs/<id>/cancel")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The job was cancelled or asked to stop.

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

Status `404`: No such job.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The job could not be updated.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
