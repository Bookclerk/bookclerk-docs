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
