# Legacy

Old unprefixed paths; use the `/api/…` route named in each.

Back to the [web API reference](README.md).

## `GET /status`

**Deprecated.**

Deprecated alias of `GET /api/status`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Library counts and daemon state. | [`StatusResponse`](schemas.md#statusresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The library could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/status' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Library counts and daemon state.

```json
{
  "accounts": 0,
  "books": 0,
  "acquired": 0,
  "pending": 0,
  "error": 0,
  "in_progress": 0,
  "listen": "string",
  "storage_backend": "string",
  "auth_enabled": true,
  "events": {
    "pending": 0,
    "running": 0,
    "suspended": 0,
    "dead_letter": 0,
    "acked": 0,
    "oldest_pending_age_secs": 0,
    "retries_total": 0,
    "suspensions_total": 0,
    "dead_letters_total": 0,
    "dispatch_latency_ms_avg": 0,
    "handler_latency_ms_avg": 0
  }
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

Status `500`: The library could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

## `POST /scan`

**Deprecated.**

Deprecated alias of `POST /api/library/scan`.

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
| 200 | The scan jobs were accepted; see `POST /api/library/scan` for the other answers. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The scan jobs were accepted; see `POST /api/library/scan` for the other answers.

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

## `POST /acquire`

**Deprecated.**

Deprecated alias of `POST /api/library/acquire`.

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
| 200 | The acquire jobs were accepted; see `POST /api/library/acquire` for the other answers. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/acquire' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The acquire jobs were accepted; see `POST /api/library/acquire` for the other answers.

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

Status `403`: The caller's role does not allow this route.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

## `GET /jobs`

**Deprecated.**

Deprecated alias of `GET /api/jobs`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Jobs, newest first. | array of [`JobInfo`](schemas.md#jobinfo) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The job table could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/jobs' \
    -H 'Authorization: Bearer <operatorToken>'
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

## `POST /jobs/{id}/cancel`

**Deprecated.**

Deprecated alias of `POST /api/jobs/{id}/cancel`.

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
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such job. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/jobs/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
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

Status `403`: The caller's role does not allow this route.

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

## `POST /integrations/{id}/scan`

Enqueues a remote integration library scan; 404 if missing, 400 if unsupported.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Integration instance id. |

### Request body

Optional.

`application/json`: [`IntegrationScanRequest`](schemas.md#integrationscanrequest) or null

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `force` | boolean or null | no | When true, the integration rescans even if it considers itself current. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The job was accepted. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 400 | The integration cannot scan a library. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No integration with this id. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 409 | The same job is already pending or running, or no node can run it. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 429 | The job queue is full. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 500 | The job could not be queued. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 503 | Job admission is paused. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/integrations/<id>/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`: The job was accepted.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `400`: The integration cannot scan a library.

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

Status `404`: No integration with this id.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `409`: The same job is already pending or running, or no node can run it.

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

Status `500`: The job could not be queued.

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
