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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The caller's open requests.

```json
[
  {
    "id": 0,
    "uuid": "string",
    "identity_id": 0,
    "title": "string",
    "authors": "string",
    "asin": "string",
    "isbn": "string",
    "notes": "string",
    "status": "open",
    "work_key": "string",
    "work_id": "string",
    "resolved_book_uuid": "string",
    "cover_url": "string",
    "sources": [
      {
        "id": 0,
        "title_request_id": 0,
        "source": "string",
        "product_id": "string",
        "title": "string",
        "subtitle": "string",
        "authors": "string",
        "narrators": "string",
        "series": "string",
        "series_index": "string",
        "asin": "string",
        "isbn": "string",
        "description": "string",
        "publisher": "string",
        "length_minutes": 0,
        "published_at": "string",
        "categories": "string",
        "language": "string",
        "cover_url": "string",
        "url": "string",
        "price_cents": 0,
        "currency": "string",
        "price_label": "string",
        "list_price_cents": 0,
        "list_price_label": "string",
        "member_price_cents": 0,
        "member_price_label": "string",
        "observed_at": "2026-01-01T12:00:00Z",
        "created_at": "2026-01-01T12:00:00Z",
        "updated_at": "2026-01-01T12:00:00Z"
      }
    ],
    "description": "string",
    "subtitle": "string",
    "narrators": "string",
    "series": "string",
    "series_index": "string",
    "publisher": "string",
    "length_minutes": 0,
    "published_at": "string",
    "genres": "string",
    "language": "string",
    "store_editions": [
      {
        "source": "string",
        "product_id": "string"
      }
    ],
    "purchase_hints": [
      {
        "source": "string",
        "product_id": "string",
        "title": "string",
        "url": "string",
        "price_cents": 0,
        "currency": "string",
        "price_label": "string",
        "list_price_cents": 0,
        "list_price_label": "string",
        "member_price_cents": 0,
        "member_price_label": "string"
      }
    ],
    "created_at": "2026-01-01T12:00:00Z",
    "updated_at": "2026-01-01T12:00:00Z"
  }
]
```

Status `401`: No signed-in caller. The body is `text/plain`.

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/wishlist`

Creates a wishlist / title-request row for the authenticated caller.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`CreateRequestBody`](schemas.md#createrequestbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | Required title text; empty values are rejected with 400. |
| `authors` | string or null | no | Optional author list as a single display string. |
| `asin` | string or null | no | Optional Audible ASIN used for work-key identity. |
| `isbn` | string or null | no | Optional ISBN used for work-key identity. |
| `notes` | string or null | no | Optional operator or member note stored on the request. |
| `store_editions` | array of [`StoreEdition`](schemas.md#storeedition) | no | Known storefront editions to persist as `title_request_sources`. |
| `purchase_hints` | array of [`PurchaseHint`](schemas.md#purchasehint) | no | Optional priced storefront links snapshotted at wishlist time. |
| `work_key` | string or null | no | Optional stable work key; when omitted one is derived from bib fields. |
| `cover_url` | string or null | no | Optional cover image URL snapshotted onto the request. |
| `description` | string or null | no | Optional plot/summary text persisted with the request. |
| `subtitle` | string or null | no | Optional subtitle stored on the request and source rows. |
| `narrators` | string or null | no | Optional narrator list as a single display string. |
| `series` | string or null | no | Optional series name. |
| `series_index` | string or null | no | Optional series position as a display string. |
| `publisher` | string or null | no | Optional publisher name. |
| `length_minutes` | integer (int64) or null (int64) | no | Optional runtime in minutes. |
| `published_at` | string or null | no | Optional publication date as a display string. |
| `genres` | string or null | no | Optional genre list as a single display string. |
| `language` | string or null | no | Optional content language code. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new request. | [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`) |
| 400 | The title is missing or invalid. | string (`text/plain`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "string"}'
```

### Example response

Status `200`: The new request.

```json
{
  "id": 0,
  "uuid": "string",
  "identity_id": 0,
  "title": "string",
  "authors": "string",
  "asin": "string",
  "isbn": "string",
  "notes": "string",
  "status": "open",
  "work_key": "string",
  "work_id": "string",
  "resolved_book_uuid": "string",
  "cover_url": "string",
  "sources": [
    {
      "id": 0,
      "title_request_id": 0,
      "source": "string",
      "product_id": "string",
      "title": "string",
      "subtitle": "string",
      "authors": "string",
      "narrators": "string",
      "series": "string",
      "series_index": "string",
      "asin": "string",
      "isbn": "string",
      "description": "string",
      "publisher": "string",
      "length_minutes": 0,
      "published_at": "string",
      "categories": "string",
      "language": "string",
      "cover_url": "string",
      "url": "string",
      "price_cents": 0,
      "currency": "string",
      "price_label": "string",
      "list_price_cents": 0,
      "list_price_label": "string",
      "member_price_cents": 0,
      "member_price_label": "string",
      "observed_at": "2026-01-01T12:00:00Z",
      "created_at": "2026-01-01T12:00:00Z",
      "updated_at": "2026-01-01T12:00:00Z"
    }
  ],
  "description": "string",
  "subtitle": "string",
  "narrators": "string",
  "series": "string",
  "series_index": "string",
  "publisher": "string",
  "length_minutes": 0,
  "published_at": "string",
  "genres": "string",
  "language": "string",
  "store_editions": [
    {
      "source": "string",
      "product_id": "string"
    }
  ],
  "purchase_hints": [
    {
      "source": "string",
      "product_id": "string",
      "title": "string",
      "url": "string",
      "price_cents": 0,
      "currency": "string",
      "price_label": "string",
      "list_price_cents": 0,
      "list_price_label": "string",
      "member_price_cents": 0,
      "member_price_label": "string"
    }
  ],
  "created_at": "2026-01-01T12:00:00Z",
  "updated_at": "2026-01-01T12:00:00Z"
}
```

Status `400`: The title is missing or invalid. The body is `text/plain`.

Status `401`: No signed-in caller. The body is `text/plain`.

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist/<uuid>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: The cancelled request.

```json
{
  "id": 0,
  "uuid": "string",
  "identity_id": 0,
  "title": "string",
  "authors": "string",
  "asin": "string",
  "isbn": "string",
  "notes": "string",
  "status": "open",
  "work_key": "string",
  "work_id": "string",
  "resolved_book_uuid": "string",
  "cover_url": "string",
  "sources": [
    {
      "id": 0,
      "title_request_id": 0,
      "source": "string",
      "product_id": "string",
      "title": "string",
      "subtitle": "string",
      "authors": "string",
      "narrators": "string",
      "series": "string",
      "series_index": "string",
      "asin": "string",
      "isbn": "string",
      "description": "string",
      "publisher": "string",
      "length_minutes": 0,
      "published_at": "string",
      "categories": "string",
      "language": "string",
      "cover_url": "string",
      "url": "string",
      "price_cents": 0,
      "currency": "string",
      "price_label": "string",
      "list_price_cents": 0,
      "list_price_label": "string",
      "member_price_cents": 0,
      "member_price_label": "string",
      "observed_at": "2026-01-01T12:00:00Z",
      "created_at": "2026-01-01T12:00:00Z",
      "updated_at": "2026-01-01T12:00:00Z"
    }
  ],
  "description": "string",
  "subtitle": "string",
  "narrators": "string",
  "series": "string",
  "series_index": "string",
  "publisher": "string",
  "length_minutes": 0,
  "published_at": "string",
  "genres": "string",
  "language": "string",
  "store_editions": [
    {
      "source": "string",
      "product_id": "string"
    }
  ],
  "purchase_hints": [
    {
      "source": "string",
      "product_id": "string",
      "title": "string",
      "url": "string",
      "price_cents": 0,
      "currency": "string",
      "price_label": "string",
      "list_price_cents": 0,
      "list_price_label": "string",
      "member_price_cents": 0,
      "member_price_label": "string"
    }
  ],
  "created_at": "2026-01-01T12:00:00Z",
  "updated_at": "2026-01-01T12:00:00Z"
}
```

Status `400`: Only open wishlist items can be removed.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `401`: No signed-in caller. The body is `text/plain`.

Status `403`: The request belongs to someone else. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. The body is `text/plain`.

Status `404`: No such request. The body is `text/plain`.

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

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

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/request-queue' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`: Open requests, ranked.

```json
[
  {
    "work_key": "string",
    "title": "string",
    "authors": "string",
    "asin": "string",
    "isbn": "string",
    "cover_url": "string",
    "wish_count": 0,
    "wishers": [
      {
        "user_id": 0,
        "identity_id": 0,
        "display_name": "string",
        "login_name": "string",
        "operator": true,
        "has_avatar": true,
        "avatar_source": "string",
        "gravatar_hash": "string",
        "picture_url": "string"
      }
    ],
    "sample_uuids": [
      "string"
    ],
    "first_requested_at": "2026-01-01T12:00:00Z",
    "last_requested_at": "2026-01-01T12:00:00Z",
    "score": 0,
    "taste_score": 0,
    "reasons": [
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

Status `500`: The queue could not be ranked. The body is `text/plain`.

Status `503`: Discover is shutting down; try again.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
