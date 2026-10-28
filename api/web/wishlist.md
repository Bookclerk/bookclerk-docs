# Wishlist

Titles users have asked for.

Back to the [web API reference](README.md).

## `GET /api/wishlist`

Lists open wishlist rows for the caller (portal identity or operator-owned).

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's open requests. Body: array of [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
  - `title` *string*, **required** — Display title of the work or edition.
  - `authors` *string or null* — Comma-separated or JSON author list from the storefront.
  - `asin` *string or null* — Amazon ASIN when known; otherwise null.
  - `isbn` *string or null* — ISBN-10/13 when known; otherwise null.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
  - `work_id` *string or null* — Canonical work id this edition or request resolves to.
  - `resolved_book_uuid` *string or null* — Library `books.uuid` once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `sources` *array of [`TitleRequestSourceRecord`](schemas.md#titlerequestsourcerecord)* — Per-storefront snapshots (empty for legacy rows).
  - `description` *string or null* — Blurb / synopsis text (may contain HTML).
  - `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
  - `narrators` *string or null* — Comma-separated or JSON narrator list when present.
  - `series` *string or null* — Series name when the title belongs to a series.
  - `series_index` *string or null* — Position within the series (storefront string form).
  - `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
  - `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
  - `published_at` *string or null* — Publication date string from the storefront or enrichment.
  - `genres` *string or null* — Merged genre / category list for display.
  - `language` *string or null* — BCP-47 or storefront language code when known.
  - `store_editions` *array of [`WishlistStoreEdition`](schemas.md#wishliststoreedition)* — Distinct storefront editions contributing to this wishlist item.
  - `purchase_hints` *array of [`WishlistPurchaseHint`](schemas.md#wishlistpurchasehint)* — Purchase URLs and prices observed per storefront.
  - `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
  - `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.
- `401` — No signed-in caller. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/wishlist' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/wishlist",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/wishlist", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/wishlist")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Personal wishlist row (also contributes to the shared global queue while open).",
    "required": [
      "id",
      "uuid",
      "title",
      "status",
      "created_at",
      "updated_at"
    ],
    "properties": {
      "id": {
        "type": "integer",
        "format": "int64",
        "description": "Surrogate primary key assigned by the database."
      },
      "uuid": {
        "type": "string",
        "description": "Stable UUID for this row (API / foreign-key identity)."
      },
      "identity_id": {
        "type": [
          "integer",
          "null"
        ],
        "format": "int64",
        "description": "Foreign key to `portal_identities.id`."
      },
      "title": {
        "type": "string",
        "description": "Display title of the work or edition."
      },
      "authors": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated or JSON author list from the storefront."
      },
      "asin": {
        "type": [
          "string",
          "null"
        ],
        "description": "Amazon ASIN when known; otherwise null."
      },
      "isbn": {
        "type": [
          "string",
          "null"
        ],
        "description": "ISBN-10/13 when known; otherwise null."
      },
      "notes": {
        "type": [
          "string",
          "null"
        ],
        "description": "Free-form operator or requester notes."
      },
      "status": {
        "type": "string",
        "description": "Lifecycle status for the row (user, request, …).",
        "enum": [
          "open",
          "cancelled"
        ]
      },
      "work_key": {
        "type": "string",
        "description": "Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation."
      },
      "work_id": {
        "type": [
          "string",
          "null"
        ],
        "description": "Canonical work id this edition or request resolves to."
      },
      "resolved_book_uuid": {
        "type": [
          "string",
          "null"
        ],
        "description": "Library `books.uuid` once the wishlist item is fulfilled."
      },
      "cover_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "HTTPS URL for cover art when known."
      },
      "sources": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "Per-storefront catalog/pricing snapshot attached to a wishlist row.",
          "required": [
            "id",
            "title_request_id",
            "source",
            "product_id",
            "created_at",
            "updated_at"
          ],
          "properties": {
            "id": {
              "type": "integer",
              "format": "int64",
              "description": "Surrogate primary key assigned by the database."
            },
            "title_request_id": {
              "type": "integer",
              "format": "int64",
              "description": "Foreign key to `title_requests.id`."
            },
            "source": {
              "type": "string",
              "description": "Content-source plugin id (`audible`, `libro`, …)."
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …)."
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title of the work or edition."
            },
            "subtitle": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional subtitle from bibliographic metadata."
            },
            "authors": {
              "type": [
                "string",
                "null"
              ],
              "description": "Comma-separated or JSON author list from the storefront."
            },
            "narrators": {
              "type": [
                "string",
                "null"
              ],
              "description": "Comma-separated or JSON narrator list when present."
            },
            "series": {
              "type": [
                "string",
                "null"
              ],
              "description": "Series name when the title belongs to a series."
            },
            "series_index": {
              "type": [
                "string",
                "null"
              ],
              "description": "Position within the series (storefront string form)."
            },
            "asin": {
              "type": [
                "string",
                "null"
              ],
              "description": "Amazon ASIN when known; otherwise null."
            },
            "isbn": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISBN-10/13 when known; otherwise null."
            },
            "description": {
              "type": [
                "string",
                "null"
              ],
              "description": "Blurb / synopsis text (may contain HTML)."
            },
            "publisher": {
              "type": [
                "string",
                "null"
              ],
              "description": "Publisher name from metadata enrichment or the storefront."
            },
            "length_minutes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Runtime in whole minutes when the storefront reports it."
            },
            "published_at": {
              "type": [
                "string",
                "null"
              ],
              "description": "Publication date string from the storefront or enrichment."
            },
            "categories": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront category / genre path list (serialized)."
            },
            "language": {
              "type": [
                "string",
                "null"
              ],
              "description": "BCP-47 or storefront language code when known."
            },
            "cover_url": {
              "type": [
                "string",
                "null"
              ],
              "description": "HTTPS URL for cover art when known."
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront product or purchase URL when known."
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Observed price in minor currency units, when known."
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields."
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted price string for display."
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "List/MSRP price in minor units, when known."
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted list price for display."
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member/subscriber price in minor units, when known."
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted member price for display."
            },
            "observed_at": {
              "type": [
                "string",
                "null"
              ],
              "format": "date-time",
              "description": "RFC 3339 time when this storefront snapshot was observed."
            },
            "created_at": {
              "type": "string",
              "format": "date-time",
              "description": "RFC 3339 timestamp when the row was inserted."
            },
            "updated_at": {
              "type": "string",
              "format": "date-time",
              "description": "RFC 3339 timestamp when the row was last modified."
            }
          }
        },
        "description": "Per-storefront snapshots (empty for legacy rows)."
      },
      "description": {
        "type": [
          "string",
          "null"
        ],
        "description": "Blurb / synopsis text (may contain HTML)."
      },
      "subtitle": {
        "type": [
          "string",
          "null"
        ],
        "description": "Optional subtitle from bibliographic metadata."
      },
      "narrators": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated or JSON narrator list when present."
      },
      "series": {
        "type": [
          "string",
          "null"
        ],
        "description": "Series name when the title belongs to a series."
      },
      "series_index": {
        "type": [
          "string",
          "null"
        ],
        "description": "Position within the series (storefront string form)."
      },
      "publisher": {
        "type": [
          "string",
          "null"
        ],
        "description": "Publisher name from metadata enrichment or the storefront."
      },
      "length_minutes": {
        "type": [
          "integer",
          "null"
        ],
        "format": "int64",
        "description": "Runtime in whole minutes when the storefront reports it."
      },
      "published_at": {
        "type": [
          "string",
          "null"
        ],
        "description": "Publication date string from the storefront or enrichment."
      },
      "genres": {
        "type": [
          "string",
          "null"
        ],
        "description": "Merged genre / category list for display."
      },
      "language": {
        "type": [
          "string",
          "null"
        ],
        "description": "BCP-47 or storefront language code when known."
      },
      "store_editions": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "Storefront edition key for wishlist / queue payloads.",
          "required": [
            "source",
            "product_id"
          ],
          "properties": {
            "source": {
              "type": "string",
              "description": "Content-source plugin id (`audible`, `libro`, …)."
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …)."
            }
          }
        },
        "description": "Distinct storefront editions contributing to this wishlist item."
      },
      "purchase_hints": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "Snapshotted purchase link/price for a wishlist title.",
          "required": [
            "source",
            "product_id"
          ],
          "properties": {
            "source": {
              "type": "string",
              "description": "Content-source plugin id (`audible`, `libro`, …)."
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …)."
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title of the work or edition."
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront product or purchase URL when known."
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Observed price in minor currency units, when known."
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields."
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted price string for display."
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "List/MSRP price in minor units, when known."
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted list price for display."
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member/subscriber price in minor units, when known."
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted member price for display."
            }
          }
        },
        "description": "Purchase URLs and prices observed per storefront."
      },
      "created_at": {
        "type": "string",
        "format": "date-time",
        "description": "RFC 3339 timestamp when the row was inserted."
      },
      "updated_at": {
        "type": "string",
        "format": "date-time",
        "description": "RFC 3339 timestamp when the row was last modified."
      }
    }
  }
}
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

- `title` *string*, **required** — Required title text; empty values are rejected with 400.
- `authors` *string or null* — Optional author list as a single display string.
- `asin` *string or null* — Optional Audible ASIN used for work-key identity.
- `isbn` *string or null* — Optional ISBN used for work-key identity.
- `notes` *string or null* — Optional operator or member note stored on the request.
- `store_editions` *array of [`StoreEdition`](schemas.md#storeedition)* — Known storefront editions to persist as `title_request_sources`.
- `purchase_hints` *array of [`PurchaseHint`](schemas.md#purchasehint)* — Optional priced storefront links snapshotted at wishlist time.
- `work_key` *string or null* — Optional stable work key; when omitted one is derived from bib fields.
- `cover_url` *string or null* — Optional cover image URL snapshotted onto the request.
- `description` *string or null* — Optional plot/summary text persisted with the request.
- `subtitle` *string or null* — Optional subtitle stored on the request and source rows.
- `narrators` *string or null* — Optional narrator list as a single display string.
- `series` *string or null* — Optional series name.
- `series_index` *string or null* — Optional series position as a display string.
- `publisher` *string or null* — Optional publisher name.
- `length_minutes` *integer (int64) or null* — Optional runtime in minutes.
- `published_at` *string or null* — Optional publication date as a display string.
- `genres` *string or null* — Optional genre list as a single display string.
- `language` *string or null* — Optional content language code.

### Responses

- `200` — The new request. Body: [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
  - `title` *string*, **required** — Display title of the work or edition.
  - `authors` *string or null* — Comma-separated or JSON author list from the storefront.
  - `asin` *string or null* — Amazon ASIN when known; otherwise null.
  - `isbn` *string or null* — ISBN-10/13 when known; otherwise null.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
  - `work_id` *string or null* — Canonical work id this edition or request resolves to.
  - `resolved_book_uuid` *string or null* — Library `books.uuid` once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `sources` *array of [`TitleRequestSourceRecord`](schemas.md#titlerequestsourcerecord)* — Per-storefront snapshots (empty for legacy rows).
  - `description` *string or null* — Blurb / synopsis text (may contain HTML).
  - `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
  - `narrators` *string or null* — Comma-separated or JSON narrator list when present.
  - `series` *string or null* — Series name when the title belongs to a series.
  - `series_index` *string or null* — Position within the series (storefront string form).
  - `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
  - `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
  - `published_at` *string or null* — Publication date string from the storefront or enrichment.
  - `genres` *string or null* — Merged genre / category list for display.
  - `language` *string or null* — BCP-47 or storefront language code when known.
  - `store_editions` *array of [`WishlistStoreEdition`](schemas.md#wishliststoreedition)* — Distinct storefront editions contributing to this wishlist item.
  - `purchase_hints` *array of [`WishlistPurchaseHint`](schemas.md#wishlistpurchasehint)* — Purchase URLs and prices observed per storefront.
  - `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
  - `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.
- `400` — The title is missing or invalid. Body: string (`text/plain`)
- `401` — No signed-in caller. Body: string (`text/plain`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/wishlist' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"title": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/wishlist",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"title": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/wishlist", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"title": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/wishlist")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"title": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "object",
  "description": "Personal wishlist row (also contributes to the shared global queue while open).",
  "required": [
    "id",
    "uuid",
    "title",
    "status",
    "created_at",
    "updated_at"
  ],
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64",
      "description": "Surrogate primary key assigned by the database."
    },
    "uuid": {
      "type": "string",
      "description": "Stable UUID for this row (API / foreign-key identity)."
    },
    "identity_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Foreign key to `portal_identities.id`."
    },
    "title": {
      "type": "string",
      "description": "Display title of the work or edition."
    },
    "authors": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON author list from the storefront."
    },
    "asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Amazon ASIN when known; otherwise null."
    },
    "isbn": {
      "type": [
        "string",
        "null"
      ],
      "description": "ISBN-10/13 when known; otherwise null."
    },
    "notes": {
      "type": [
        "string",
        "null"
      ],
      "description": "Free-form operator or requester notes."
    },
    "status": {
      "type": "string",
      "description": "Lifecycle status for the row (user, request, …).",
      "enum": [
        "open",
        "cancelled"
      ]
    },
    "work_key": {
      "type": "string",
      "description": "Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation."
    },
    "work_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Canonical work id this edition or request resolves to."
    },
    "resolved_book_uuid": {
      "type": [
        "string",
        "null"
      ],
      "description": "Library `books.uuid` once the wishlist item is fulfilled."
    },
    "cover_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "HTTPS URL for cover art when known."
    },
    "sources": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Per-storefront catalog/pricing snapshot attached to a wishlist row.",
        "required": [
          "id",
          "title_request_id",
          "source",
          "product_id",
          "created_at",
          "updated_at"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64",
            "description": "Surrogate primary key assigned by the database."
          },
          "title_request_id": {
            "type": "integer",
            "format": "int64",
            "description": "Foreign key to `title_requests.id`."
          },
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition."
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle from bibliographic metadata."
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON author list from the storefront."
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON narrator list when present."
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a series."
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (storefront string form)."
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Amazon ASIN when known; otherwise null."
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISBN-10/13 when known; otherwise null."
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Blurb / synopsis text (may contain HTML)."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher name from metadata enrichment or the storefront."
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in whole minutes when the storefront reports it."
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the storefront or enrichment."
          },
          "categories": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront category / genre path list (serialized)."
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 or storefront language code when known."
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when known."
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known."
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known."
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields."
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display."
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known."
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display."
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known."
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display."
          },
          "observed_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 time when this storefront snapshot was observed."
          },
          "created_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was inserted."
          },
          "updated_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was last modified."
          }
        }
      },
      "description": "Per-storefront snapshots (empty for legacy rows)."
    },
    "description": {
      "type": [
        "string",
        "null"
      ],
      "description": "Blurb / synopsis text (may contain HTML)."
    },
    "subtitle": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional subtitle from bibliographic metadata."
    },
    "narrators": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON narrator list when present."
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "Series name when the title belongs to a series."
    },
    "series_index": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position within the series (storefront string form)."
    },
    "publisher": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publisher name from metadata enrichment or the storefront."
    },
    "length_minutes": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in whole minutes when the storefront reports it."
    },
    "published_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publication date string from the storefront or enrichment."
    },
    "genres": {
      "type": [
        "string",
        "null"
      ],
      "description": "Merged genre / category list for display."
    },
    "language": {
      "type": [
        "string",
        "null"
      ],
      "description": "BCP-47 or storefront language code when known."
    },
    "store_editions": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Storefront edition key for wishlist / queue payloads.",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          }
        }
      },
      "description": "Distinct storefront editions contributing to this wishlist item."
    },
    "purchase_hints": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Snapshotted purchase link/price for a wishlist title.",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition."
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known."
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known."
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields."
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display."
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known."
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display."
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known."
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display."
          }
        }
      },
      "description": "Purchase URLs and prices observed per storefront."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was inserted."
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was last modified."
    }
  }
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

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/wishlist/{uuid}`

Cancels an open wishlist item the caller owns; others get 403.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

- `uuid` (path) *string*, **required** — Request UUID.

### Responses

- `200` — The cancelled request. Body: [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
  - `title` *string*, **required** — Display title of the work or edition.
  - `authors` *string or null* — Comma-separated or JSON author list from the storefront.
  - `asin` *string or null* — Amazon ASIN when known; otherwise null.
  - `isbn` *string or null* — ISBN-10/13 when known; otherwise null.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation.
  - `work_id` *string or null* — Canonical work id this edition or request resolves to.
  - `resolved_book_uuid` *string or null* — Library `books.uuid` once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `sources` *array of [`TitleRequestSourceRecord`](schemas.md#titlerequestsourcerecord)* — Per-storefront snapshots (empty for legacy rows).
  - `description` *string or null* — Blurb / synopsis text (may contain HTML).
  - `subtitle` *string or null* — Optional subtitle from bibliographic metadata.
  - `narrators` *string or null* — Comma-separated or JSON narrator list when present.
  - `series` *string or null* — Series name when the title belongs to a series.
  - `series_index` *string or null* — Position within the series (storefront string form).
  - `publisher` *string or null* — Publisher name from metadata enrichment or the storefront.
  - `length_minutes` *integer (int64) or null* — Runtime in whole minutes when the storefront reports it.
  - `published_at` *string or null* — Publication date string from the storefront or enrichment.
  - `genres` *string or null* — Merged genre / category list for display.
  - `language` *string or null* — BCP-47 or storefront language code when known.
  - `store_editions` *array of [`WishlistStoreEdition`](schemas.md#wishliststoreedition)* — Distinct storefront editions contributing to this wishlist item.
  - `purchase_hints` *array of [`WishlistPurchaseHint`](schemas.md#wishlistpurchasehint)* — Purchase URLs and prices observed per storefront.
  - `created_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was inserted.
  - `updated_at` *string (date-time)*, **required** — RFC 3339 timestamp when the row was last modified.
- `400` — Only open wishlist items can be removed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `401` — No signed-in caller. Body: string (`text/plain`)
- `403` — The request belongs to someone else. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: string (`text/plain`)
- `404` — No such request. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist/<uuid>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/wishlist/<uuid>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/wishlist/<uuid>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/wishlist/<uuid>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/wishlist/<uuid>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "object",
  "description": "Personal wishlist row (also contributes to the shared global queue while open).",
  "required": [
    "id",
    "uuid",
    "title",
    "status",
    "created_at",
    "updated_at"
  ],
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64",
      "description": "Surrogate primary key assigned by the database."
    },
    "uuid": {
      "type": "string",
      "description": "Stable UUID for this row (API / foreign-key identity)."
    },
    "identity_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Foreign key to `portal_identities.id`."
    },
    "title": {
      "type": "string",
      "description": "Display title of the work or edition."
    },
    "authors": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON author list from the storefront."
    },
    "asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Amazon ASIN when known; otherwise null."
    },
    "isbn": {
      "type": [
        "string",
        "null"
      ],
      "description": "ISBN-10/13 when known; otherwise null."
    },
    "notes": {
      "type": [
        "string",
        "null"
      ],
      "description": "Free-form operator or requester notes."
    },
    "status": {
      "type": "string",
      "description": "Lifecycle status for the row (user, request, …).",
      "enum": [
        "open",
        "cancelled"
      ]
    },
    "work_key": {
      "type": "string",
      "description": "Stable bibliographic key (`isbn:…` / `asin:…` / `soft:…`) for aggregation."
    },
    "work_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Canonical work id this edition or request resolves to."
    },
    "resolved_book_uuid": {
      "type": [
        "string",
        "null"
      ],
      "description": "Library `books.uuid` once the wishlist item is fulfilled."
    },
    "cover_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "HTTPS URL for cover art when known."
    },
    "sources": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Per-storefront catalog/pricing snapshot attached to a wishlist row.",
        "required": [
          "id",
          "title_request_id",
          "source",
          "product_id",
          "created_at",
          "updated_at"
        ],
        "properties": {
          "id": {
            "type": "integer",
            "format": "int64",
            "description": "Surrogate primary key assigned by the database."
          },
          "title_request_id": {
            "type": "integer",
            "format": "int64",
            "description": "Foreign key to `title_requests.id`."
          },
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition."
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle from bibliographic metadata."
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON author list from the storefront."
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON narrator list when present."
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a series."
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (storefront string form)."
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Amazon ASIN when known; otherwise null."
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISBN-10/13 when known; otherwise null."
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Blurb / synopsis text (may contain HTML)."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher name from metadata enrichment or the storefront."
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in whole minutes when the storefront reports it."
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the storefront or enrichment."
          },
          "categories": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront category / genre path list (serialized)."
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 or storefront language code when known."
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when known."
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known."
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known."
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields."
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display."
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known."
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display."
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known."
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display."
          },
          "observed_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 time when this storefront snapshot was observed."
          },
          "created_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was inserted."
          },
          "updated_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was last modified."
          }
        }
      },
      "description": "Per-storefront snapshots (empty for legacy rows)."
    },
    "description": {
      "type": [
        "string",
        "null"
      ],
      "description": "Blurb / synopsis text (may contain HTML)."
    },
    "subtitle": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional subtitle from bibliographic metadata."
    },
    "narrators": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON narrator list when present."
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "Series name when the title belongs to a series."
    },
    "series_index": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position within the series (storefront string form)."
    },
    "publisher": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publisher name from metadata enrichment or the storefront."
    },
    "length_minutes": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in whole minutes when the storefront reports it."
    },
    "published_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publication date string from the storefront or enrichment."
    },
    "genres": {
      "type": [
        "string",
        "null"
      ],
      "description": "Merged genre / category list for display."
    },
    "language": {
      "type": [
        "string",
        "null"
      ],
      "description": "BCP-47 or storefront language code when known."
    },
    "store_editions": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Storefront edition key for wishlist / queue payloads.",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          }
        }
      },
      "description": "Distinct storefront editions contributing to this wishlist item."
    },
    "purchase_hints": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Snapshotted purchase link/price for a wishlist title.",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …)."
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …)."
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition."
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known."
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known."
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields."
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display."
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known."
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display."
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known."
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display."
          }
        }
      },
      "description": "Purchase URLs and prices observed per storefront."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was inserted."
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was last modified."
    }
  }
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

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
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

- `200` — Open requests, ranked. Body: array of [`RankedQueueEntry`](schemas.md#rankedqueueentry) (`application/json`)
  - `work_key` *string*, **required** — Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
  - `title` *string*, **required** — Display title as shown on the storefront or library card.
  - `authors` *string or null* — Comma-separated author names when the storefront provides them.
  - `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
  - `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
  - `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
  - `wish_count` *integer (int64)*, **required** — Number of distinct users who wishlisted this title.
  - `wishers` *array of [`QueueWisher`](schemas.md#queuewisher)* — Distinct people who have this title on an open wishlist.
  - `sample_uuids` *array of string*, **required** — Sample Bookclerk user UUIDs who requested this title (privacy-capped).
  - `first_requested_at` *string (date-time)*, **required** — UTC timestamp of the earliest wishlist request for this work.
  - `last_requested_at` *string (date-time)*, **required** — UTC timestamp of the latest wishlist request for this work.
  - `score` *number (double)*, **required** — Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`]).
  - `taste_score` *number (double)*, **required** — Local taste / embedding component before the wish-count boost.
  - `reasons` *array of string*, **required** — Short human-readable explanations for why this item was ranked here.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The queue could not be ranked. Body: string (`text/plain`)
- `503` — Discover is shutting down; try again. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/request-queue' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/request-queue' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/request-queue",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/request-queue", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/request-queue")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
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

Schema:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "description": "Global wishlist queue entry ranked by overall/operator recommend taste +\nheavy wish-count weight (shared order; not per-viewer personalized).",
    "required": [
      "work_key",
      "title",
      "wish_count",
      "sample_uuids",
      "first_requested_at",
      "last_requested_at",
      "score",
      "taste_score",
      "reasons"
    ],
    "properties": {
      "work_key": {
        "type": "string",
        "description": "Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author)."
      },
      "title": {
        "type": "string",
        "description": "Display title as shown on the storefront or library card."
      },
      "authors": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated author names when the storefront provides them."
      },
      "asin": {
        "type": [
          "string",
          "null"
        ],
        "description": "Audible / Amazon ASIN when this edition is sold on Audible."
      },
      "isbn": {
        "type": [
          "string",
          "null"
        ],
        "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published."
      },
      "cover_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "HTTPS URL for cover art when the catalog exposes one."
      },
      "wish_count": {
        "type": "integer",
        "format": "int64",
        "description": "Number of distinct users who wishlisted this title."
      },
      "wishers": {
        "type": "array",
        "items": {
          "type": "object",
          "description": "Compact wisher shown on the global queue (avatar + label).",
          "properties": {
            "user_id": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "First-party `users.id` when the portal identity is linked."
            },
            "identity_id": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Portal identity that created the wish (dedupe for unlinked identities)."
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display name, identity label, or `\"Operator\"`."
            },
            "login_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Local login name when the first-party user has one."
            },
            "operator": {
              "type": "boolean",
              "description": "True when the wish came from the operator token (no portal user)."
            },
            "has_avatar": {
              "type": "boolean",
              "description": "True when an uploaded avatar file exists (filled by the daemon)."
            },
            "avatar_source": {
              "type": [
                "string",
                "null"
              ],
              "description": "Explicit picture choice when the user has set one."
            },
            "gravatar_hash": {
              "type": [
                "string",
                "null"
              ],
              "description": "SHA-256 hex of the contact email for Gravatar."
            },
            "picture_url": {
              "type": [
                "string",
                "null"
              ],
              "description": "HTTPS picture from the wishing identity, when any."
            }
          }
        },
        "description": "Distinct people who have this title on an open wishlist."
      },
      "sample_uuids": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Sample Bookclerk user UUIDs who requested this title (privacy-capped)."
      },
      "first_requested_at": {
        "type": "string",
        "format": "date-time",
        "description": "UTC timestamp of the earliest wishlist request for this work."
      },
      "last_requested_at": {
        "type": "string",
        "format": "date-time",
        "description": "UTC timestamp of the latest wishlist request for this work."
      },
      "score": {
        "type": "number",
        "format": "double",
        "description": "Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`])."
      },
      "taste_score": {
        "type": "number",
        "format": "double",
        "description": "Local taste / embedding component before the wish-count boost."
      },
      "reasons": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Short human-readable explanations for why this item was ranked here."
      }
    }
  }
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

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
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

Schema:

```json
{
  "type": "object",
  "description": "An error. `error` is a stable slug; bodies the error middleware writes also carry `message` and `status`, and some routes add fields of their own.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Stable machine slug."
    },
    "message": {
      "type": "string",
      "description": "Explanation for people."
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated."
    }
  }
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
