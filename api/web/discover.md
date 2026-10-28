# Discover

Recommendations, catalog search and title metadata.

Back to the [web API reference](README.md).

## `POST /api/discover/sync-listening`

Enqueues a listening-progress sync; 400 when no capable integrations are loaded.

**Access:** The operator, or a signed-in Administrator, Owner or user allowed to acquire.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The job was accepted. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `400` — No listening-capable integration is enabled. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — The same job is already pending or running, or no node can run it. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `429` — The job queue is full. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `500` — The job could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `503` — Job admission is paused. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/sync-listening' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/sync-listening' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/sync-listening",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/sync-listening", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/sync-listening")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Status `400`: No listening-capable integration is enabled. The body is `text/plain`.

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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/discover/recommendations`

Builds the Discover recommendation feed, skipping embed work on an empty library.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `limit` (query) *integer* — Maximum recommendations to return.
- `user` (query) *string* — Optional external user id used to bias the feed.
- `no_purchase_hints` (query) *boolean* — When true, skip storefront price/availability enrichment.
- `no_listening` (query) *boolean* — When true, ignore listening_progress (owned-library taste only).
- `listening_providers` (query) *string* — Comma-separated integration ids; empty = all listening providers.

### Responses

- `200` — The Discover feed. Body: [`DiscoverFeed`](schemas.md#discoverfeed) (`application/json`)
  - `shelves` *array of [`DiscoverShelf`](schemas.md#discovershelf)*, **required** — Ordered horizontal shelves that make up the Discover page.
  - `shelf_kinds` *array of [`ShelfKindInfo`](schemas.md#shelfkindinfo)* — Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on).
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `503` — Discover is shutting down; try again. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/recommendations' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/recommendations' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/discover/recommendations",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/recommendations", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/discover/recommendations")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The Discover feed.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `503`: Discover is shutting down; try again.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/discover/purchase-hints`

Resolves storefront price/availability for one title, forcing server-side preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)

- `title` *string*, **required** — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `candidate_source` *string or null* — Storefront id of the edition that produced this card.
- `candidate_product_id` *string or null* — Storefront product id of the edition that produced this card.
- `store_editions` *array of [`StoreEdition`](schemas.md#storeedition)* — Known storefront editions already on the recommendation card.
- `region` *string or null* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `preferred_sources` *array of string* — Storefronts the caller has linked accounts for. Treated as **member** pricing when picking `best`; other stores are compared at list / non-member price. Every catalog match is still returned in `hints`.

### Responses

- `200` — Prices and availability. Body: [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse) (`application/json`)
  - `hints` *array of [`PurchaseHint`](schemas.md#purchasehint)*, **required** — Resolved purchase hints for this query.
  - `best` *[`PurchaseHint`](schemas.md#purchasehint) or null* — Lowest-priced hint (or first catalog hit when no prices resolved).
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/purchase-hints' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"title": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/purchase-hints",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"title": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/purchase-hints", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"title": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/purchase-hints")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"title": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Prices and availability.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/discover/purchase-hints/batch`

Resolves up to 24 purchase-hint queries with caller-derived preferred stores.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PurchaseHintsBatchBody`](schemas.md#purchasehintsbatchbody)

- `queries` *array of [`PurchaseHintsQuery`](schemas.md#purchasehintsquery)* — Per-title purchase-hint queries in this batch.

### Responses

- `200` — One result per query, in order. Body: [`PurchaseHintsBatchResponse`](schemas.md#purchasehintsbatchresponse) (`application/json`)
  - `results` *array of [`PurchaseHintsResponse`](schemas.md#purchasehintsresponse)*, **required** — One purchase-hint response per query, in request order.
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/purchase-hints/batch' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/purchase-hints/batch",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/purchase-hints/batch", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/purchase-hints/batch")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One result per query, in order.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/discover/search`

Runs a multi-store catalog search with a 15s wall budget and empty-query short circuit.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `q` (query) *string* — Search text; queries shorter than two characters return an empty page. Rejected above 256 UTF-8 bytes as a downstream catalog-search limit (the HTTP query extractor has already materialized `q`).
- `limit` (query) *integer* — Alias for [`Self::page_size`] (typeahead / legacy).
- `page_size` (query) *integer* — Page size (1–48); falls back to `limit` then 24.
- `cursor` (query) *string* — Opaque cursor from a previous [`bookclerk_discover::CatalogSearchPage`].
- `sort` (query) *string* — `relevance` / `popularity` / `rating` / `title` / `author` / `price` / `length`.
- `sort_dir` (query) *string* — `asc` / `desc` (defaults per sort when omitted).
- `region` (query) *string* — Storefront region code (default `us`).
- `field` (query) *string* — Optional facet scope: `author` / `narrator` / `series` / `genre`.
- `lang` (query) *string* — Preferred / default content language (BCP-47 primary). When `language` is omitted and `all_languages` is not set, applied as a hard include.
- `language` (query) *string* — Comma-separated language codes to include (`en`, `zh`, …).
- `all_languages` (query) *boolean* — When true, do not hard-filter by language (soft prefer via `lang` only).
- `author` (query) *string* — Comma-separated include filters (OR within a kind).
- `narrator` (query) *string* — Comma-separated narrator include filters.
- `series` (query) *string* — Comma-separated series include filters.
- `genre` (query) *string* — Comma-separated genre include filters.
- `source` (query) *string* — Comma-separated store ids to include.
- `exclude_source` (query) *string* — Comma-separated store ids to exclude.
- `exclude_narrator` (query) *string* — Comma-separated narrator substrings to exclude (e.g. `Virtual Voice`).
- `min_rating` (query) *number (double)* — Minimum overall rating (0–5); missing ratings still pass.
- `min_length_minutes` (query) *integer (int64)* — Minimum runtime in minutes; values ≤ 0 are ignored.
- `max_length_minutes` (query) *integer (int64)* — Maximum runtime in minutes; values ≤ 0 are ignored.

### Responses

- `200` — A page of catalog results. Body: [`CatalogSearchPage`](schemas.md#catalogsearchpage) (`application/json`)
  - `items` *array of [`CatalogSearchHit`](schemas.md#catalogsearchhit)*, **required** — Ordered collection of child entries for this response.
  - `page_size` *integer*, **required** — Maximum number of items returned in this page. At least 0.
  - `has_more` *boolean*, **required** — When true, another page is available via [`Self::next_cursor`].
  - `next_cursor` *string or null* — Opaque cursor for the next page; omit or `None` when exhausted.
  - `sort` *string*, **required** — Sort field applied to the merged result set.
  - `sort_dir` *string* — Ascending or descending order for [`Self::sort`].
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/search' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/search' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/discover/search",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/search", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/discover/search")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of catalog results.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/discover/title-meta`

Resolves Audnexus-style title metadata for one query.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaQuery`](schemas.md#titlemetaquery)

- `title` *string* — Display title as shown on the storefront or library card.
- `authors` *string or null* — Comma-separated author names when the storefront provides them.
- `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
- `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
- `narrators` *string or null* — Comma-separated narrator names when known.
- `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.

### Responses

- `200` — The metadata, or null when nothing matched. Body: [`TitleMeta`](schemas.md#titlemeta) or null (`application/json`)
  - `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
  - `title` *string or null* — Display title as shown on the storefront or library card.
  - `subtitle` *string or null* — Optional subtitle when the catalog distinguishes it from the title.
  - `authors` *string or null* — Comma-separated author names when the storefront provides them.
  - `narrators` *string or null* — Comma-separated narrator names when known.
  - `series` *string or null* — Series name when the title belongs to a named series.
  - `series_index` *string or null* — Position within the series (e.g. `1`, `1.5`) when known.
  - `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
  - `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
  - `description` *string or null* — Publisher or storefront synopsis; may be truncated for embeddings.
  - `publisher` *string or null* — Publisher imprint as reported by the catalog.
  - `length_minutes` *integer (int64) or null* — Audiobook runtime in whole minutes when known.
  - `published_at` *string or null* — Publication date string from the catalog (ISO or storefront format).
  - `categories` *string or null* — Genre / subject tags as a single delimited string when known.
  - `language` *string or null* — BCP-47 / storefront language code (`en`, `de`, …) when known.
  - `is_abridged` *boolean or null* — `Some(true)` abridged / `Some(false)` unabridged when the storefront said so.
  - `rating_overall` *number (double) or null* — Audible community overall rating (0–5).
  - `rating_performance` *number (double) or null* — Narration / performance sub-score when the storefront splits ratings.
  - `rating_story` *number (double) or null* — Story / content sub-score when the storefront splits ratings.
  - `rating_count` *integer (int64) or null* — Count of star ratings (Audible `num_ratings`).
  - `review_count` *integer (int64) or null* — Count of written reviews (Audible `num_reviews`).
  - `reviews` *array of [`TitleReview`](schemas.md#titlereview)* — Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty for older clients; prefer the paginated reviews endpoint.
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-meta' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-meta",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-meta", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-meta")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The metadata, or null when nothing matched.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/discover/title-meta/batch`

Resolves up to 24 title-metadata queries without failing the whole batch on one error.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleMetaBatchBody`](schemas.md#titlemetabatchbody)

- `queries` *array of [`TitleMetaQuery`](schemas.md#titlemetaquery)* — Per-title metadata queries in this batch.

### Responses

- `200` — One result per query, in order. Body: [`TitleMetaBatchResponse`](schemas.md#titlemetabatchresponse) (`application/json`)
  - `results` *array of [`TitleMeta`](schemas.md#titlemeta) or null*, **required** — One optional metadata payload per query; enrichment errors become `None`.
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-meta/batch' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-meta/batch' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-meta/batch",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-meta/batch", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-meta/batch")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: One result per query, in order.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/discover/title-reviews`

Fetches Audnexus reviews for an ASIN; rejects an empty ASIN with 400.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`TitleReviewsQuery`](schemas.md#titlereviewsquery)

- `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
- `region` *string* — Marketplace / region code (`us`, `uk`, …) for catalog lookups.
- `page` *integer (int32)* — 1-based page index. At least 0.
- `page_size` *integer (int32)* — Page size (clamped server-side to 1..=20). At least 0.
- `sort_by` *string* — `MostHelpful` (default) or `MostRecent`.

### Responses

- `200` — A page of reviews. Body: [`TitleReviewsPage`](schemas.md#titlereviewspage) (`application/json`)
  - `asin` *string*, **required** — Audible / Amazon ASIN when this edition is sold on Audible.
  - `page` *integer (int32)*, **required** — 1-based page index for this reviews / list request. At least 0.
  - `page_size` *integer (int32)*, **required** — Maximum number of items returned in this page. At least 0.
  - `has_more` *boolean*, **required** — When true, another page is available by incrementing [`Self::page`].
  - `sort_by` *string*, **required** — Remote sort key for reviews (`most_helpful`, `most_recent`, …).
  - `reviews` *array of [`TitleReview`](schemas.md#titlereview)*, **required** — Review rows on this page.
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The lookup failed. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-reviews' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"asin": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-reviews' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"asin": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-reviews",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"asin": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-reviews", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"asin": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-reviews")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"asin": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of reviews.

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

Status `400`: The query is malformed or too large. The body is `text/plain`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The lookup failed. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
