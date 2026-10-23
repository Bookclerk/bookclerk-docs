# Wishlist

Titles users have asked for.

Back to the [web API reference](README.md).

## `GET /api/wishlist`

Lists open wishlist rows for the caller (portal identity or operator-owned).

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The caller's open requests. | array of [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/wishlist`

Creates a wishlist / title-request row for the authenticated caller.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`CreateRequestBody`](schemas.md#createrequestbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new request. | [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`) |
| 400 | The title is missing or invalid. | string (`text/plain`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/wishlist/{uuid}`

Cancels an open wishlist item the caller owns; others get 403.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `uuid` | path | string | yes | Request UUID. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The cancelled request. | [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`) |
| 400 | Only open wishlist items can be removed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 403 | The request belongs to someone else. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | string (`text/plain`) |
| 404 | No such request. | string (`text/plain`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/request-queue`

Ranks the shared open-request queue without warming ONNX embeddings.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Open requests, ranked. | array of [`RankedQueueEntry`](schemas.md#rankedqueueentry) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The queue could not be ranked. | string (`text/plain`) |
| 503 | Discover is shutting down; try again. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
