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

## `POST /api/discover/purchase-hints`

Resolves storefront price/availability for one title, forcing server-side preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Prices and availability. | [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/discover/purchase-hints/batch`

Resolves up to 24 purchase-hint queries with caller-derived preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsBatchBody`](schemas.md#purchasehintsbatchbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | One result per query, in order. | [`PurchaseHintsBatchResponse`](schemas.md#purchasehintsbatchresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

## `POST /api/discover/title-meta`

Resolves Audnexus-style title metadata for one query.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaQuery`](schemas.md#titlemetaquery)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The metadata, or null when nothing matched. | [`TitleMeta`](schemas.md#titlemeta) or null (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/discover/title-meta/batch`

Resolves up to 24 title-metadata queries without failing the whole batch on one error.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaBatchBody`](schemas.md#titlemetabatchbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | One result per query, in order. | [`TitleMetaBatchResponse`](schemas.md#titlemetabatchresponse) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/discover/title-reviews`

Fetches Audnexus reviews for an ASIN; rejects an empty ASIN with 400.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleReviewsQuery`](schemas.md#titlereviewsquery)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | A page of reviews. | [`TitleReviewsPage`](schemas.md#titlereviewspage) (`application/json`) |
| 400 | The query is malformed or too large. | string (`text/plain`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The lookup failed. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
