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

## `POST /scan`

**Deprecated.**

Deprecated alias of `POST /api/library/scan`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`ScanRequest`](schemas.md#scanrequest) or null

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The scan jobs were accepted; see `POST /api/library/scan` for the other answers. | [`ScanPlanResponse`](schemas.md#scanplanresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |

## `POST /acquire`

**Deprecated.**

Deprecated alias of `POST /api/library/acquire`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

Optional.

`application/json`: [`AcquireRequestBody`](schemas.md#acquirerequestbody) or null

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The acquire jobs were accepted; see `POST /api/library/acquire` for the other answers. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |

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
