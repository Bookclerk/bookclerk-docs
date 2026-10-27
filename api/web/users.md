# Users

Library users and their claim tickets.

Back to the [web API reference](README.md).

## `GET /api/users`

List first-party users (operator or administrator provisioner).

Includes presence extras: non-expired portal sessions (`online` /
`last_active_at`), durable `last_seen_at` (survives logout), unfinished
listening within the last 30 minutes, and linked storefront accounts.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The library's users. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/users`

Owner/admin or operator creates a user and optional invite magic link.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`CreateUserRequest`](schemas.md#createuserrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `role` | string or null | no | Role to assign when permitted (`member` when omitted). |
| `display_name` | string or null | no | Optional display name for the new user. |
| `login_name` | string or null | no | Optional local login name for the new user. |
| `email` | string or null | no | Optional email for the new user. |
| `password` | string or null | no | Optional password; hashed before persist. Empty or omitted leaves the user passwordless. |
| `mint_invite` | boolean | no | When true (default), also mint an invite/claim ticket. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new user, with an invite link when one was asked for. | any JSON (`application/json`) |
| 400 | The request is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The login is taken. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `DELETE /api/users/{id}`

Delete a user and their personal data (wishlist / links / sessions); keep acquired books.

Operators/administrators may delete any user. Portal callers may delete
only their own account (self-service).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | User id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The user and their personal data were deleted. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The user is the last Owner or Administrator. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

## `PATCH /api/users/{id}`

Patch role/status/display/login for a first-party user (provisioner).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | User id. |

### Request body

`application/json`: [`PatchUserRequest`](schemas.md#patchuserrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `role` | string or null | no | Replacement role (`owner` / `administrator` / `member`) when present. |
| `status` | string or null | no | Replacement status (`active` / `disabled`) when present. |
| `display_name` | string or null | no | Replacement display name when present. |
| `login_name` | string or null | no | Replacement local login name when present. |
| `email` | string or null | no | Replacement email when present. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The user after the change. | any JSON (`application/json`) |
| 400 | A value is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The login name or email is taken by another user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `POST /api/users/{id}/claim-ticket`

Mint a fresh local claim ticket for an active first-party user.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | User id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A fresh claim ticket. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/claim-ticket' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/users/{id}/reset-password`

Invalidate a user's password, revoke sessions, and mint a claim ticket for reset.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | User id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The password was cleared; a claim ticket for the reset. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such user. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/reset-password' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

## `GET /api/users/{id}/avatar`

Serves a stored avatar JPEG for any authenticated household member.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | integer (int64) | yes | User id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The avatar. | [`BinaryBody`](schemas.md#binarybody) (`image/jpeg`); [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/webp`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The user has no avatar. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/users/<id>/avatar' \
    -H 'Authorization: Bearer <operatorToken>'
```
