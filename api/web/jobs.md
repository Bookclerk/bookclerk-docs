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
