# Discover

Recommendations, catalog search and title metadata.

Back to the [web API reference](README.md).

## `POST /api/discover/sync-listening`

Enqueues a listening-progress sync; 400 when no capable integrations are loaded.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The job was accepted. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 400 | No listening-capable integration is enabled. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The same job is already pending or running, or no node can run it. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 429 | The job queue is full. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 500 | The job could not be queued. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 503 | Job admission is paused. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/sync-listening' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`:

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

## `GET /api/discover/recommendations`

Builds the Discover recommendation feed, skipping embed work on an empty library.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `limit` | query | integer | no | Maximum recommendations to return. |
| `user` | query | string | no | Optional external user id used to bias the feed. |
| `no_purchase_hints` | query | boolean | no | When true, skip storefront price/availability enrichment. |
| `no_listening` | query | boolean | no | When true, ignore listening_progress (owned-library taste only). |
| `listening_providers` | query | string | no | Comma-separated integration ids; empty = all listening providers. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The Discover feed. | [`DiscoverFeed`](schemas.md#discoverfeed) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 503 | Discover is shutting down; try again. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/recommendations' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`:

```json
{
  "shelves": [
    {
      "id": "string",
      "title": "string",
      "subtitle": "string",
      "items": [
        {
          "title": "string",
          "score": 0,
          "reasons": [],
          "purchase_hints": [],
          "from_request": true
        }
      ]
    }
  ],
  "shelf_kinds": [
    {
      "id": "string",
      "label": "string"
    }
  ]
}
```

## `POST /api/discover/purchase-hints`

Resolves storefront price/availability for one title, forcing server-side preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `candidate_source` | string or null | no | Storefront id of the edition that produced this card. |
| `candidate_product_id` | string or null | no | Storefront product id of the edition that produced this card. |
| `store_editions` | array of [`StoreEdition`](schemas.md#storeedition) | no | Known storefront editions already on the recommendation card. |
| `region` | string or null | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |
| `preferred_sources` | array of string | no | Storefronts the caller has linked accounts for. Treated as **member** pricing when picking `best`; other stores are compared at list / non-member price. Every catalog match is still returned in `hints`. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Prices and availability. | [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "string"}'
```

### Example response

Status `200`:

```json
{
  "hints": [
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
  "best": {
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
}
```

## `POST /api/discover/purchase-hints/batch`

Resolves up to 24 purchase-hint queries with caller-derived preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsBatchBody`](schemas.md#purchasehintsbatchbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `queries` | array of [`PurchaseHintsQuery`](schemas.md#purchasehintsquery) | no | Per-title purchase-hint queries in this batch. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | One result per query, in order. | [`PurchaseHintsBatchResponse`](schemas.md#purchasehintsbatchresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`:

```json
{
  "results": [
    {
      "hints": [
        {
          "source": "string",
          "product_id": "string"
        }
      ],
      "best": {
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
    }
  ]
}
```

## `GET /api/discover/search`

Runs a multi-store catalog search with a 15s wall budget and empty-query short circuit.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `q` | query | string | no | Search text; queries shorter than two characters return an empty page. Rejected above 256 UTF-8 bytes as a downstream catalog-search limit (the HTTP query extractor has already materialized `q`). |
| `limit` | query | integer | no | Alias for [`Self::page_size`] (typeahead / legacy). |
| `page_size` | query | integer | no | Page size (1–48); falls back to `limit` then 24. |
| `cursor` | query | string | no | Opaque cursor from a previous [`bookclerk_discover::CatalogSearchPage`]. |
| `sort` | query | string | no | `relevance` / `popularity` / `rating` / `title` / `author` / `price` / `length`. |
| `sort_dir` | query | string | no | `asc` / `desc` (defaults per sort when omitted). |
| `region` | query | string | no | Storefront region code (default `us`). |
| `field` | query | string | no | Optional facet scope: `author` / `narrator` / `series` / `genre`. |
| `lang` | query | string | no | Preferred / default content language (BCP-47 primary). When `language` is omitted and `all_languages` is not set, applied as a hard include. |
| `language` | query | string | no | Comma-separated language codes to include (`en`, `zh`, …). |
| `all_languages` | query | boolean | no | When true, do not hard-filter by language (soft prefer via `lang` only). |
| `author` | query | string | no | Comma-separated include filters (OR within a kind). |
| `narrator` | query | string | no | Comma-separated narrator include filters. |
| `series` | query | string | no | Comma-separated series include filters. |
| `genre` | query | string | no | Comma-separated genre include filters. |
| `source` | query | string | no | Comma-separated store ids to include. |
| `exclude_source` | query | string | no | Comma-separated store ids to exclude. |
| `exclude_narrator` | query | string | no | Comma-separated narrator substrings to exclude (e.g. `Virtual Voice`). |
| `min_rating` | query | number (double) | no | Minimum overall rating (0–5); missing ratings still pass. |
| `min_length_minutes` | query | integer (int64) | no | Minimum runtime in minutes; values ≤ 0 are ignored. |
| `max_length_minutes` | query | integer (int64) | no | Maximum runtime in minutes; values ≤ 0 are ignored. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A page of catalog results. | [`CatalogSearchPage`](schemas.md#catalogsearchpage) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/search' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`:

```json
{
  "items": [
    {
      "work_key": "string",
      "title": "string",
      "authors": "string",
      "narrators": "string",
      "series": "string",
      "series_index": "string",
      "asin": "string",
      "isbn": "string",
      "cover_url": "string",
      "store_editions": [
        {
          "source": "string",
          "product_id": "string"
        }
      ],
      "sources": [
        "string"
      ],
      "subtitle": "string",
      "description": "string",
      "publisher": "string",
      "length_minutes": 0,
      "published_at": "string",
      "genres": "string",
      "language": "string",
      "is_abridged": true,
      "rating_overall": 0,
      "rating_count": 0,
      "price_cents": 0,
      "purchase_hints": [
        {
          "source": "string",
          "product_id": "string"
        }
      ]
    }
  ],
  "page_size": 0,
  "has_more": true,
  "next_cursor": "string",
  "sort": "string",
  "sort_dir": "string"
}
```

## `POST /api/discover/title-meta`

Resolves Audnexus-style title metadata for one query.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaQuery`](schemas.md#titlemetaquery)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | no | Display title as shown on the storefront or library card. |
| `authors` | string or null | no | Comma-separated author names when the storefront provides them. |
| `asin` | string or null | no | Audible / Amazon ASIN when this edition is sold on Audible. |
| `isbn` | string or null | no | Canonical ISBN-13 (or ISBN-10 normalized) when published. |
| `narrators` | string or null | no | Comma-separated narrator names when known. |
| `length_minutes` | integer (int64) or null (int64) | no | Audiobook runtime in whole minutes when known. |
| `region` | string | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The metadata, or null when nothing matched. | [`TitleMeta`](schemas.md#titlemeta) or null (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`:

```json
{
  "asin": "string",
  "title": "string",
  "subtitle": "string",
  "authors": "string",
  "narrators": "string",
  "series": "string",
  "series_index": "string",
  "isbn": "string",
  "cover_url": "string",
  "description": "string",
  "publisher": "string",
  "length_minutes": 0,
  "published_at": "string",
  "categories": "string",
  "language": "string",
  "is_abridged": true,
  "rating_overall": 0,
  "rating_performance": 0,
  "rating_story": 0,
  "rating_count": 0,
  "review_count": 0,
  "reviews": [
    {
      "id": "string",
      "title": "string",
      "body": "string",
      "author_name": "string",
      "overall_rating": 0,
      "performance_rating": 0,
      "story_rating": 0,
      "submitted_at": "string"
    }
  ]
}
```

## `POST /api/discover/title-meta/batch`

Resolves up to 24 title-metadata queries without failing the whole batch on one error.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaBatchBody`](schemas.md#titlemetabatchbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `queries` | array of [`TitleMetaQuery`](schemas.md#titlemetaquery) | no | Per-title metadata queries in this batch. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | One result per query, in order. | [`TitleMetaBatchResponse`](schemas.md#titlemetabatchresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`:

```json
{
  "results": [
    {
      "asin": "string",
      "title": "string",
      "subtitle": "string",
      "authors": "string",
      "narrators": "string",
      "series": "string",
      "series_index": "string",
      "isbn": "string",
      "cover_url": "string",
      "description": "string",
      "publisher": "string",
      "length_minutes": 0,
      "published_at": "string",
      "categories": "string",
      "language": "string",
      "is_abridged": true,
      "rating_overall": 0,
      "rating_performance": 0,
      "rating_story": 0,
      "rating_count": 0,
      "review_count": 0,
      "reviews": [
        {
          "body": "string"
        }
      ]
    }
  ]
}
```

## `POST /api/discover/title-reviews`

Fetches Audnexus reviews for an ASIN; rejects an empty ASIN with 400.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleReviewsQuery`](schemas.md#titlereviewsquery)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `asin` | string | yes | Audible / Amazon ASIN when this edition is sold on Audible. |
| `region` | string | no | Marketplace / region code (`us`, `uk`, …) for catalog lookups. |
| `page` | integer (int32) | no | 1-based page index. At least 0. |
| `page_size` | integer (int32) | no | Page size (clamped server-side to 1..=20). At least 0. |
| `sort_by` | string | no | `MostHelpful` (default) or `MostRecent`. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A page of reviews. | [`TitleReviewsPage`](schemas.md#titlereviewspage) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-reviews' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"asin": "string"}'
```

### Example response

Status `200`:

```json
{
  "asin": "string",
  "page": 0,
  "page_size": 0,
  "has_more": true,
  "sort_by": "string",
  "reviews": [
    {
      "id": "string",
      "title": "string",
      "body": "string",
      "author_name": "string",
      "overall_rating": 0,
      "performance_rating": 0,
      "story_rating": 0,
      "submitted_at": "string"
    }
  ]
}
```
