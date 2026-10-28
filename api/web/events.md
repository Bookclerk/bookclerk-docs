# Events

Domain events and their deliveries.

Back to the [web API reference](README.md).

## `GET /api/events`

Returns recent domain-event outbox rows, newest first.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Recent domain events, newest first. | array of [`DomainEventInfo`](schemas.md#domaineventinfo) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The outbox could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Recent domain events, newest first.

```json
[
  {
    "id": "string",
    "eventType": "string",
    "schemaVersion": 0,
    "dispatchState": "string",
    "accountId": "string",
    "dedupKey": "string",
    "createdAt": "string",
    "queueName": "string",
    "contentType": "string",
    "source": "string",
    "deliverAfter": "string"
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

Status `500`: The outbox could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/events/deliveries`

Returns event deliveries, optionally filtered by `state`.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `state` | query | string | no | Delivery state (`dead_letter`, `pending`, …). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Deliveries, newest first. | array of [`EventDeliveryInfo`](schemas.md#eventdeliveryinfo) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The outbox could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Deliveries, newest first.

```json
[
  {
    "id": "string",
    "eventId": "string",
    "pluginId": "string",
    "state": "string",
    "attemptCount": 0,
    "leaseGeneration": 0,
    "invocationSequence": 0,
    "outcome": "string",
    "errorMessage": "string",
    "runAfter": "string",
    "updatedAt": "string",
    "cancelRequested": true,
    "resourceClass": "string",
    "resumePending": true,
    "hasCheckpoint": true,
    "queueName": "string"
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

Status `500`: The outbox could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/queues`

Returns every queue the outbox or the live catalog knows about, with
delivery counters and the consumers subscribed to it.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Every known queue. | array of [`QueueInfo`](schemas.md#queueinfo) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The outbox could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/queues' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Every known queue.

```json
[
  {
    "name": "string",
    "core": true,
    "pending": 0,
    "running": 0,
    "acked": 0,
    "deadLetter": 0,
    "consumers": [
      "string"
    ]
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

Status `500`: The outbox could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/queues/{name}`

Returns one queue's counters, consumers, recent messages, and deliveries.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `name` | path | string | yes | Queue name. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The queue. | [`QueueDetail`](schemas.md#queuedetail) (`application/json`) |
| 400 | The queue name is invalid. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The outbox could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/queues/<name>' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The queue.

```json
{
  "name": "string",
  "core": true,
  "pending": 0,
  "running": 0,
  "acked": 0,
  "deadLetter": 0,
  "consumers": [
    "string"
  ],
  "messages": [
    {
      "id": "string",
      "eventType": "string",
      "schemaVersion": 0,
      "dispatchState": "string",
      "accountId": "string",
      "dedupKey": "string",
      "createdAt": "string",
      "queueName": "string",
      "contentType": "string",
      "source": "string",
      "deliverAfter": "string"
    }
  ],
  "deliveries": [
    {
      "id": "string",
      "eventId": "string",
      "pluginId": "string",
      "state": "string",
      "attemptCount": 0,
      "leaseGeneration": 0,
      "invocationSequence": 0,
      "outcome": "string",
      "errorMessage": "string",
      "runAfter": "string",
      "updatedAt": "string",
      "cancelRequested": true,
      "resourceClass": "string",
      "resumePending": true,
      "hasCheckpoint": true,
      "queueName": "string"
    }
  ]
}
```

Status `400`: The queue name is invalid.

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

Status `500`: The outbox could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/events/deliveries/{id}/retry`

Re-queue a dead-lettered delivery.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Delivery id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The delivery was queued again. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such delivery, or it is not in a state this applies to. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The outbox could not be read or written. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/retry' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The delivery was queued again.

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

Status `404`: No such delivery, or it is not in a state this applies to.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The outbox could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/events/deliveries/{id}/acknowledge`

Acknowledge a dead-lettered delivery (no further retry).

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Delivery id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The delivery was acknowledged. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such delivery, or it is not in a state this applies to. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The outbox could not be read or written. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/acknowledge' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The delivery was acknowledged.

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

Status `404`: No such delivery, or it is not in a state this applies to.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The outbox could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/events/deliveries/{id}/cancel`

Cancel a pending delivery or request cooperative stop of a running one.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Delivery id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The delivery was cancelled or asked to stop. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such delivery, or it is not in a state this applies to. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The outbox could not be read or written. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/cancel' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The delivery was cancelled or asked to stop.

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

Status `404`: No such delivery, or it is not in a state this applies to.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The outbox could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/events/deliveries/{id}/resume`

Resume a suspended delivery (wake now, next claim uses resume_pending).

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Delivery id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The delivery was resumed. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such delivery, or it is not in a state this applies to. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The outbox could not be read or written. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/events/deliveries/<id>/resume' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The delivery was resumed.

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

Status `404`: No such delivery, or it is not in a state this applies to.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The outbox could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
