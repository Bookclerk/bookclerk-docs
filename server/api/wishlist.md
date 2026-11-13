# Wishlist

Titles users have asked for.

Back to the [web API reference](README.md).

## `GET /api/wishlist`

Lists open wishlist rows for the caller (portal identity or operator-owned).

**Access:** The operator, or any library session, including a client sign-in's token (refused with 403 `transport_insecure` off loopback without TLS, or `client_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clientToken`

### Responses

- `200` — The caller's open requests. Body: array of [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
  - `title` *string*, **required** — Display title: the title of the wish's work.
  - `authors` *string or null* — Comma-separated authors: the work's author credits in billing order.
  - `asin` *string or null* — Amazon ASIN from the wish's storefront sources, when one carries it.
  - `isbn` *string or null* — ISBN-10/13 from the wish's storefront sources, when one carries it.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use to match a wish to a catalog card. Derived from the work's title and authors and the sources' identifiers on every read, never stored.
  - `work_id` *string*, **required** — The work the wish is for; wishes for one work are one queue entry.
  - `edition_id` *string or null* — The edition the wish named, when the library knows it.
  - `resolved_edition_id` *string or null* — Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `401` — No signed-in caller. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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
    "id": 31,
    "uuid": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c",
    "identity_id": 7,
    "title": "The Left Hand of Darkness",
    "authors": "Ursula K. Le Guin",
    "asin": "B002V0QK4C",
    "isbn": "9780441478125",
    "notes": "Book club pick for March",
    "status": "open",
    "work_key": "isbn:9780441478125",
    "work_id": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c",
    "edition_id": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b",
    "resolved_edition_id": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
    "cover_url": "https://images.example.com/covers/9780441478125.jpg",
    "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
    "sources": [
      {
        "id": 88,
        "title_request_id": 31,
        "source": "libro",
        "product_id": "9780441478125",
        "title": "The Left Hand of Darkness",
        "subtitle": "A Hainish Novel",
        "authors": "Ursula K. Le Guin",
        "narrators": "George Guidall",
        "series": "Hainish Cycle",
        "series_index": "4",
        "asin": "B002V0QK4C",
        "isbn": "9780441478125",
        "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
        "publisher": "Blackstone Publishing",
        "length_minutes": 560,
        "published_at": "2005-06-21",
        "categories": "Science Fiction & Fantasy > Science Fiction",
        "language": "en",
        "cover_url": "https://images.example.com/covers/9780441478125.jpg",
        "url": "https://libro.fm/audiobooks/9780441478125",
        "price_cents": 2499,
        "currency": "USD",
        "price_label": "$24.99",
        "list_price_cents": 2999,
        "list_price_label": "$29.99",
        "member_price_cents": 1499,
        "member_price_label": "$14.99",
        "observed_at": "2026-10-29T10:12:00Z",
        "created_at": "2026-10-29T10:12:00Z",
        "updated_at": "2026-10-29T10:14:22Z"
      }
    ],
    "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
    "subtitle": "A Hainish Novel",
    "narrators": "George Guidall",
    "series": "Hainish Cycle",
    "series_index": "4",
    "publisher": "Blackstone Publishing",
    "length_minutes": 560,
    "published_at": "2005-06-21",
    "genres": "Science Fiction, Classics",
    "language": "en",
    "store_editions": [
      {
        "source": "libro",
        "product_id": "9780441478125"
      }
    ],
    "purchase_hints": [
      {
        "source": "libro",
        "product_id": "9780441478125",
        "title": "The Left Hand of Darkness",
        "url": "https://libro.fm/audiobooks/9780441478125",
        "price_cents": 2499,
        "currency": "USD",
        "price_label": "$24.99",
        "list_price_cents": 2999,
        "list_price_label": "$29.99",
        "member_price_cents": 1499,
        "member_price_label": "$14.99"
      }
    ],
    "created_at": "2026-10-29T10:12:00Z",
    "updated_at": "2026-10-29T10:14:22Z"
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
      "work_id",
      "created_at",
      "updated_at"
    ],
    "properties": {
      "id": {
        "type": "integer",
        "format": "int64",
        "description": "Surrogate primary key assigned by the database.",
        "example": 31
      },
      "uuid": {
        "type": "string",
        "description": "Stable UUID for this row (API / foreign-key identity).",
        "example": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c"
      },
      "identity_id": {
        "type": [
          "integer",
          "null"
        ],
        "format": "int64",
        "description": "Foreign key to `portal_identities.id`.",
        "example": 7
      },
      "title": {
        "type": "string",
        "description": "Display title: the title of the wish's work.",
        "example": "The Left Hand of Darkness"
      },
      "authors": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated authors: the work's author credits in billing order.",
        "example": "Ursula K. Le Guin"
      },
      "asin": {
        "type": [
          "string",
          "null"
        ],
        "description": "Amazon ASIN from the wish's storefront sources, when one carries it.",
        "example": "B002V0QK4C"
      },
      "isbn": {
        "type": [
          "string",
          "null"
        ],
        "description": "ISBN-10/13 from the wish's storefront sources, when one carries it.",
        "example": "9780441478125"
      },
      "notes": {
        "type": [
          "string",
          "null"
        ],
        "description": "Free-form operator or requester notes.",
        "example": "Book club pick for March"
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
        "description": "Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use\nto match a wish to a catalog card. Derived from the work's title and\nauthors and the sources' identifiers on every read, never stored.",
        "example": "isbn:9780441478125"
      },
      "work_id": {
        "type": "string",
        "description": "The work the wish is for; wishes for one work are one queue entry.",
        "example": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c"
      },
      "edition_id": {
        "type": [
          "string",
          "null"
        ],
        "description": "The edition the wish named, when the library knows it.",
        "example": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b"
      },
      "resolved_edition_id": {
        "type": [
          "string",
          "null"
        ],
        "description": "Edition (`editions.id`) the library holds once the wishlist item is fulfilled.",
        "example": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
      },
      "cover_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "HTTPS URL for cover art when known.",
        "example": "https://images.example.com/covers/9780441478125.jpg"
      },
      "cover_path": {
        "type": [
          "string",
          "null"
        ],
        "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
        "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
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
              "description": "Surrogate primary key assigned by the database.",
              "example": 88
            },
            "title_request_id": {
              "type": "integer",
              "format": "int64",
              "description": "Foreign key to `title_requests.id`.",
              "example": 31
            },
            "source": {
              "type": "string",
              "description": "Content-source plugin id (`audible`, `libro`, …).",
              "example": "libro"
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …).",
              "example": "9780441478125"
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title of the work or edition.",
              "example": "The Left Hand of Darkness"
            },
            "subtitle": {
              "type": [
                "string",
                "null"
              ],
              "description": "Optional subtitle from bibliographic metadata.",
              "example": "A Hainish Novel"
            },
            "authors": {
              "type": [
                "string",
                "null"
              ],
              "description": "Comma-separated or JSON author list from the storefront.",
              "example": "Ursula K. Le Guin"
            },
            "narrators": {
              "type": [
                "string",
                "null"
              ],
              "description": "Comma-separated or JSON narrator list when present.",
              "example": "George Guidall"
            },
            "series": {
              "type": [
                "string",
                "null"
              ],
              "description": "Series name when the title belongs to a series.",
              "example": "Hainish Cycle"
            },
            "series_index": {
              "type": [
                "string",
                "null"
              ],
              "description": "Position within the series (storefront string form).",
              "example": "4"
            },
            "asin": {
              "type": [
                "string",
                "null"
              ],
              "description": "Amazon ASIN when known; otherwise null.",
              "example": "B002V0QK4C"
            },
            "isbn": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISBN-10/13 when known; otherwise null.",
              "example": "9780441478125"
            },
            "description": {
              "type": [
                "string",
                "null"
              ],
              "description": "Blurb / synopsis text (may contain HTML).",
              "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
            },
            "publisher": {
              "type": [
                "string",
                "null"
              ],
              "description": "Publisher name from metadata enrichment or the storefront.",
              "example": "Blackstone Publishing"
            },
            "length_minutes": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Runtime in whole minutes when the storefront reports it.",
              "example": 560
            },
            "published_at": {
              "type": [
                "string",
                "null"
              ],
              "description": "Publication date string from the storefront or enrichment.",
              "example": "2005-06-21"
            },
            "categories": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront category / genre path list (serialized).",
              "example": "Science Fiction & Fantasy > Science Fiction"
            },
            "language": {
              "type": [
                "string",
                "null"
              ],
              "description": "BCP-47 or storefront language code when known.",
              "example": "en"
            },
            "cover_url": {
              "type": [
                "string",
                "null"
              ],
              "description": "HTTPS URL for cover art when known.",
              "example": "https://images.example.com/covers/9780441478125.jpg"
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront product or purchase URL when known.",
              "example": "https://libro.fm/audiobooks/9780441478125"
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Observed price in minor currency units, when known.",
              "example": 2499
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields.",
              "example": "USD"
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted price string for display.",
              "example": "$24.99"
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "List/MSRP price in minor units, when known.",
              "example": 2999
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted list price for display.",
              "example": "$29.99"
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member/subscriber price in minor units, when known.",
              "example": 1499
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted member price for display.",
              "example": "$14.99"
            },
            "observed_at": {
              "type": [
                "string",
                "null"
              ],
              "format": "date-time",
              "description": "RFC 3339 time when this storefront snapshot was observed.",
              "example": "2026-10-29T10:12:00Z"
            },
            "created_at": {
              "type": "string",
              "format": "date-time",
              "description": "RFC 3339 timestamp when the row was inserted.",
              "example": "2026-10-29T10:12:00Z"
            },
            "updated_at": {
              "type": "string",
              "format": "date-time",
              "description": "RFC 3339 timestamp when the row was last modified.",
              "example": "2026-10-29T10:14:22Z"
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
        "description": "Blurb / synopsis text (may contain HTML).",
        "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
      },
      "subtitle": {
        "type": [
          "string",
          "null"
        ],
        "description": "Optional subtitle from bibliographic metadata.",
        "example": "A Hainish Novel"
      },
      "narrators": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated or JSON narrator list when present.",
        "example": "George Guidall"
      },
      "series": {
        "type": [
          "string",
          "null"
        ],
        "description": "Series name when the title belongs to a series.",
        "example": "Hainish Cycle"
      },
      "series_index": {
        "type": [
          "string",
          "null"
        ],
        "description": "Position within the series (storefront string form).",
        "example": "4"
      },
      "publisher": {
        "type": [
          "string",
          "null"
        ],
        "description": "Publisher name from metadata enrichment or the storefront.",
        "example": "Blackstone Publishing"
      },
      "length_minutes": {
        "type": [
          "integer",
          "null"
        ],
        "format": "int64",
        "description": "Runtime in whole minutes when the storefront reports it.",
        "example": 560
      },
      "published_at": {
        "type": [
          "string",
          "null"
        ],
        "description": "Publication date string from the storefront or enrichment.",
        "example": "2005-06-21"
      },
      "genres": {
        "type": [
          "string",
          "null"
        ],
        "description": "Merged genre / category list for display.",
        "example": "Science Fiction, Classics"
      },
      "language": {
        "type": [
          "string",
          "null"
        ],
        "description": "BCP-47 or storefront language code when known.",
        "example": "en"
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
              "description": "Content-source plugin id (`audible`, `libro`, …).",
              "example": "libro"
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …).",
              "example": "9780441478125"
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
              "description": "Content-source plugin id (`audible`, `libro`, …).",
              "example": "libro"
            },
            "product_id": {
              "type": "string",
              "description": "Storefront product id (ASIN, ISBN, UUID, …).",
              "example": "9780441478125"
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title of the work or edition.",
              "example": "The Left Hand of Darkness"
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront product or purchase URL when known.",
              "example": "https://libro.fm/audiobooks/9780441478125"
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Observed price in minor currency units, when known.",
              "example": 2499
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields.",
              "example": "USD"
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted price string for display.",
              "example": "$24.99"
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "List/MSRP price in minor units, when known.",
              "example": 2999
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted list price for display.",
              "example": "$29.99"
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member/subscriber price in minor units, when known.",
              "example": 1499
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Storefront-formatted member price for display.",
              "example": "$14.99"
            }
          }
        },
        "description": "Purchase URLs and prices observed per storefront."
      },
      "created_at": {
        "type": "string",
        "format": "date-time",
        "description": "RFC 3339 timestamp when the row was inserted.",
        "example": "2026-10-29T10:12:00Z"
      },
      "updated_at": {
        "type": "string",
        "format": "date-time",
        "description": "RFC 3339 timestamp when the row was last modified.",
        "example": "2026-10-29T10:14:22Z"
      }
    }
  }
}
```

Status `401`: No signed-in caller.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`.

```json
{
  "error": "client_sign_in_unavailable",
  "message": "This library does not accept client apps on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The request could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/wishlist`

Creates a wishlist / title-request row for the authenticated caller.

**Access:** The operator, or any library session, including a client sign-in's token (refused with 403 `transport_insecure` off loopback without TLS, or `client_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clientToken`

### Request body

`application/json`: [`CreateRequestBody`](schemas.md#createrequestbody)

- `title` *string*, **required** — Required title text; empty values are rejected with 400.
- `authors` *string or null* — Optional author list as a single display string.
- `asin` *string or null* — Optional Audible ASIN used for work-key identity.
- `isbn` *string or null* — Optional ISBN used for work-key identity.
- `notes` *string or null* — Optional operator or member note stored on the request.
- `store_editions` *array of [`StoreEdition`](schemas.md#storeedition)* — Known storefront editions to persist as `title_request_sources`.
- `purchase_hints` *array of [`PurchaseHint`](schemas.md#purchasehint)* — Optional priced storefront links snapshotted at wishlist time.
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
  - `title` *string*, **required** — Display title: the title of the wish's work.
  - `authors` *string or null* — Comma-separated authors: the work's author credits in billing order.
  - `asin` *string or null* — Amazon ASIN from the wish's storefront sources, when one carries it.
  - `isbn` *string or null* — ISBN-10/13 from the wish's storefront sources, when one carries it.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use to match a wish to a catalog card. Derived from the work's title and authors and the sources' identifiers on every read, never stored.
  - `work_id` *string*, **required** — The work the wish is for; wishes for one work are one queue entry.
  - `edition_id` *string or null* — The edition the wish named, when the library knows it.
  - `resolved_edition_id` *string or null* — Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `400` — The title is missing or invalid. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — No signed-in caller. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/wishlist' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "The Left Hand of Darkness"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/wishlist' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"title": "The Left Hand of Darkness"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/wishlist",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"title": "The Left Hand of Darkness"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/wishlist", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"title": "The Left Hand of Darkness"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/wishlist")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"title": "The Left Hand of Darkness"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new request.

```json
{
  "id": 31,
  "uuid": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c",
  "identity_id": 7,
  "title": "The Left Hand of Darkness",
  "authors": "Ursula K. Le Guin",
  "asin": "B002V0QK4C",
  "isbn": "9780441478125",
  "notes": "Book club pick for March",
  "status": "open",
  "work_key": "isbn:9780441478125",
  "work_id": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c",
  "edition_id": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b",
  "resolved_edition_id": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
  "cover_url": "https://images.example.com/covers/9780441478125.jpg",
  "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
  "sources": [
    {
      "id": 88,
      "title_request_id": 31,
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "subtitle": "A Hainish Novel",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "asin": "B002V0QK4C",
      "isbn": "9780441478125",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "categories": "Science Fiction & Fantasy > Science Fiction",
      "language": "en",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 2499,
      "currency": "USD",
      "price_label": "$24.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99",
      "observed_at": "2026-10-29T10:12:00Z",
      "created_at": "2026-10-29T10:12:00Z",
      "updated_at": "2026-10-29T10:14:22Z"
    }
  ],
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "subtitle": "A Hainish Novel",
  "narrators": "George Guidall",
  "series": "Hainish Cycle",
  "series_index": "4",
  "publisher": "Blackstone Publishing",
  "length_minutes": 560,
  "published_at": "2005-06-21",
  "genres": "Science Fiction, Classics",
  "language": "en",
  "store_editions": [
    {
      "source": "libro",
      "product_id": "9780441478125"
    }
  ],
  "purchase_hints": [
    {
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 2499,
      "currency": "USD",
      "price_label": "$24.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99"
    }
  ],
  "created_at": "2026-10-29T10:12:00Z",
  "updated_at": "2026-10-29T10:14:22Z"
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
    "work_id",
    "created_at",
    "updated_at"
  ],
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64",
      "description": "Surrogate primary key assigned by the database.",
      "example": 31
    },
    "uuid": {
      "type": "string",
      "description": "Stable UUID for this row (API / foreign-key identity).",
      "example": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c"
    },
    "identity_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Foreign key to `portal_identities.id`.",
      "example": 7
    },
    "title": {
      "type": "string",
      "description": "Display title: the title of the wish's work.",
      "example": "The Left Hand of Darkness"
    },
    "authors": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated authors: the work's author credits in billing order.",
      "example": "Ursula K. Le Guin"
    },
    "asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Amazon ASIN from the wish's storefront sources, when one carries it.",
      "example": "B002V0QK4C"
    },
    "isbn": {
      "type": [
        "string",
        "null"
      ],
      "description": "ISBN-10/13 from the wish's storefront sources, when one carries it.",
      "example": "9780441478125"
    },
    "notes": {
      "type": [
        "string",
        "null"
      ],
      "description": "Free-form operator or requester notes.",
      "example": "Book club pick for March"
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
      "description": "Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use\nto match a wish to a catalog card. Derived from the work's title and\nauthors and the sources' identifiers on every read, never stored.",
      "example": "isbn:9780441478125"
    },
    "work_id": {
      "type": "string",
      "description": "The work the wish is for; wishes for one work are one queue entry.",
      "example": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c"
    },
    "edition_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "The edition the wish named, when the library knows it.",
      "example": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b"
    },
    "resolved_edition_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Edition (`editions.id`) the library holds once the wishlist item is fulfilled.",
      "example": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
    },
    "cover_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "HTTPS URL for cover art when known.",
      "example": "https://images.example.com/covers/9780441478125.jpg"
    },
    "cover_path": {
      "type": [
        "string",
        "null"
      ],
      "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
      "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
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
            "description": "Surrogate primary key assigned by the database.",
            "example": 88
          },
          "title_request_id": {
            "type": "integer",
            "format": "int64",
            "description": "Foreign key to `title_requests.id`.",
            "example": 31
          },
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition.",
            "example": "The Left Hand of Darkness"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle from bibliographic metadata.",
            "example": "A Hainish Novel"
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON author list from the storefront.",
            "example": "Ursula K. Le Guin"
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON narrator list when present.",
            "example": "George Guidall"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a series.",
            "example": "Hainish Cycle"
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (storefront string form).",
            "example": "4"
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Amazon ASIN when known; otherwise null.",
            "example": "B002V0QK4C"
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISBN-10/13 when known; otherwise null.",
            "example": "9780441478125"
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Blurb / synopsis text (may contain HTML).",
            "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher name from metadata enrichment or the storefront.",
            "example": "Blackstone Publishing"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in whole minutes when the storefront reports it.",
            "example": 560
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the storefront or enrichment.",
            "example": "2005-06-21"
          },
          "categories": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront category / genre path list (serialized).",
            "example": "Science Fiction & Fantasy > Science Fiction"
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 or storefront language code when known.",
            "example": "en"
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when known.",
            "example": "https://images.example.com/covers/9780441478125.jpg"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known.",
            "example": 2499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields.",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display.",
            "example": "$24.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display.",
            "example": "$14.99"
          },
          "observed_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 time when this storefront snapshot was observed.",
            "example": "2026-10-29T10:12:00Z"
          },
          "created_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was inserted.",
            "example": "2026-10-29T10:12:00Z"
          },
          "updated_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was last modified.",
            "example": "2026-10-29T10:14:22Z"
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
      "description": "Blurb / synopsis text (may contain HTML).",
      "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
    },
    "subtitle": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional subtitle from bibliographic metadata.",
      "example": "A Hainish Novel"
    },
    "narrators": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON narrator list when present.",
      "example": "George Guidall"
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "Series name when the title belongs to a series.",
      "example": "Hainish Cycle"
    },
    "series_index": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position within the series (storefront string form).",
      "example": "4"
    },
    "publisher": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publisher name from metadata enrichment or the storefront.",
      "example": "Blackstone Publishing"
    },
    "length_minutes": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in whole minutes when the storefront reports it.",
      "example": 560
    },
    "published_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publication date string from the storefront or enrichment.",
      "example": "2005-06-21"
    },
    "genres": {
      "type": [
        "string",
        "null"
      ],
      "description": "Merged genre / category list for display.",
      "example": "Science Fiction, Classics"
    },
    "language": {
      "type": [
        "string",
        "null"
      ],
      "description": "BCP-47 or storefront language code when known.",
      "example": "en"
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
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
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
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition.",
            "example": "The Left Hand of Darkness"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known.",
            "example": 2499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields.",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display.",
            "example": "$24.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display.",
            "example": "$14.99"
          }
        }
      },
      "description": "Purchase URLs and prices observed per storefront."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was inserted.",
      "example": "2026-10-29T10:12:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was last modified.",
      "example": "2026-10-29T10:14:22Z"
    }
  }
}
```

Status `400`: The title is missing or invalid. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: No signed-in caller.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "client_sign_in_unavailable",
  "message": "This library does not accept client apps on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `title` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The request could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `DELETE /api/wishlist/{uuid}`

Cancels an open wishlist item the caller owns; others get 403.

**Access:** The operator, or any library session, including a client sign-in's token (refused with 403 `transport_insecure` off loopback without TLS, or `client_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clientToken`

### Parameters

- `uuid` (path) *string*, **required** — Request UUID.

### Responses

- `200` — The cancelled request. Body: [`TitleRequestRecord`](schemas.md#titlerequestrecord) (`application/json`)
  - `id` *integer (int64)*, **required** — Surrogate primary key assigned by the database.
  - `uuid` *string*, **required** — Stable UUID for this row (API / foreign-key identity).
  - `identity_id` *integer (int64) or null* — Foreign key to `portal_identities.id`.
  - `title` *string*, **required** — Display title: the title of the wish's work.
  - `authors` *string or null* — Comma-separated authors: the work's author credits in billing order.
  - `asin` *string or null* — Amazon ASIN from the wish's storefront sources, when one carries it.
  - `isbn` *string or null* — ISBN-10/13 from the wish's storefront sources, when one carries it.
  - `notes` *string or null* — Free-form operator or requester notes.
  - `status` *[`RequestStatus`](schemas.md#requeststatus)*, **required** — Lifecycle status for the row (user, request, …).
  - `work_key` *string* — Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use to match a wish to a catalog card. Derived from the work's title and authors and the sources' identifiers on every read, never stored.
  - `work_id` *string*, **required** — The work the wish is for; wishes for one work are one queue entry.
  - `edition_id` *string or null* — The edition the wish named, when the library knows it.
  - `resolved_edition_id` *string or null* — Edition (`editions.id`) the library holds once the wishlist item is fulfilled.
  - `cover_url` *string or null* — HTTPS URL for cover art when known.
  - `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
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
- `400` — Only open wishlist items can be removed. Body: string (`text/plain`)
- `401` — No signed-in caller. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The request belongs to someone else. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: string (`text/plain`); [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No such request. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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
  "id": 31,
  "uuid": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c",
  "identity_id": 7,
  "title": "The Left Hand of Darkness",
  "authors": "Ursula K. Le Guin",
  "asin": "B002V0QK4C",
  "isbn": "9780441478125",
  "notes": "Book club pick for March",
  "status": "open",
  "work_key": "isbn:9780441478125",
  "work_id": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c",
  "edition_id": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b",
  "resolved_edition_id": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31",
  "cover_url": "https://images.example.com/covers/9780441478125.jpg",
  "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
  "sources": [
    {
      "id": 88,
      "title_request_id": 31,
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "subtitle": "A Hainish Novel",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "asin": "B002V0QK4C",
      "isbn": "9780441478125",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "categories": "Science Fiction & Fantasy > Science Fiction",
      "language": "en",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 2499,
      "currency": "USD",
      "price_label": "$24.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99",
      "observed_at": "2026-10-29T10:12:00Z",
      "created_at": "2026-10-29T10:12:00Z",
      "updated_at": "2026-10-29T10:14:22Z"
    }
  ],
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "subtitle": "A Hainish Novel",
  "narrators": "George Guidall",
  "series": "Hainish Cycle",
  "series_index": "4",
  "publisher": "Blackstone Publishing",
  "length_minutes": 560,
  "published_at": "2005-06-21",
  "genres": "Science Fiction, Classics",
  "language": "en",
  "store_editions": [
    {
      "source": "libro",
      "product_id": "9780441478125"
    }
  ],
  "purchase_hints": [
    {
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 2499,
      "currency": "USD",
      "price_label": "$24.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99"
    }
  ],
  "created_at": "2026-10-29T10:12:00Z",
  "updated_at": "2026-10-29T10:14:22Z"
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
    "work_id",
    "created_at",
    "updated_at"
  ],
  "properties": {
    "id": {
      "type": "integer",
      "format": "int64",
      "description": "Surrogate primary key assigned by the database.",
      "example": 31
    },
    "uuid": {
      "type": "string",
      "description": "Stable UUID for this row (API / foreign-key identity).",
      "example": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c"
    },
    "identity_id": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Foreign key to `portal_identities.id`.",
      "example": 7
    },
    "title": {
      "type": "string",
      "description": "Display title: the title of the wish's work.",
      "example": "The Left Hand of Darkness"
    },
    "authors": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated authors: the work's author credits in billing order.",
      "example": "Ursula K. Le Guin"
    },
    "asin": {
      "type": [
        "string",
        "null"
      ],
      "description": "Amazon ASIN from the wish's storefront sources, when one carries it.",
      "example": "B002V0QK4C"
    },
    "isbn": {
      "type": [
        "string",
        "null"
      ],
      "description": "ISBN-10/13 from the wish's storefront sources, when one carries it.",
      "example": "9780441478125"
    },
    "notes": {
      "type": [
        "string",
        "null"
      ],
      "description": "Free-form operator or requester notes.",
      "example": "Book club pick for March"
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
      "description": "Bibliographic card key (`isbn:…` / `asin:…` / `soft:…`) clients use\nto match a wish to a catalog card. Derived from the work's title and\nauthors and the sources' identifiers on every read, never stored.",
      "example": "isbn:9780441478125"
    },
    "work_id": {
      "type": "string",
      "description": "The work the wish is for; wishes for one work are one queue entry.",
      "example": "7c1e9a2b-3d4f-4e5a-9b6c-8d7e0f1a2b3c"
    },
    "edition_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "The edition the wish named, when the library knows it.",
      "example": "0b8e3f2a-5c1d-5e7f-9a2b-4c6d8e0f1a3b"
    },
    "resolved_edition_id": {
      "type": [
        "string",
        "null"
      ],
      "description": "Edition (`editions.id`) the library holds once the wishlist item is fulfilled.",
      "example": "3f2c0d7e-9d4b-4a62-8b1e-6d0f7c2a9b31"
    },
    "cover_url": {
      "type": [
        "string",
        "null"
      ],
      "description": "HTTPS URL for cover art when known.",
      "example": "https://images.example.com/covers/9780441478125.jpg"
    },
    "cover_path": {
      "type": [
        "string",
        "null"
      ],
      "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
      "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
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
            "description": "Surrogate primary key assigned by the database.",
            "example": 88
          },
          "title_request_id": {
            "type": "integer",
            "format": "int64",
            "description": "Foreign key to `title_requests.id`.",
            "example": 31
          },
          "source": {
            "type": "string",
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition.",
            "example": "The Left Hand of Darkness"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle from bibliographic metadata.",
            "example": "A Hainish Novel"
          },
          "authors": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON author list from the storefront.",
            "example": "Ursula K. Le Guin"
          },
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated or JSON narrator list when present.",
            "example": "George Guidall"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a series.",
            "example": "Hainish Cycle"
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (storefront string form).",
            "example": "4"
          },
          "asin": {
            "type": [
              "string",
              "null"
            ],
            "description": "Amazon ASIN when known; otherwise null.",
            "example": "B002V0QK4C"
          },
          "isbn": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISBN-10/13 when known; otherwise null.",
            "example": "9780441478125"
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Blurb / synopsis text (may contain HTML).",
            "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher name from metadata enrichment or the storefront.",
            "example": "Blackstone Publishing"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Runtime in whole minutes when the storefront reports it.",
            "example": 560
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the storefront or enrichment.",
            "example": "2005-06-21"
          },
          "categories": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront category / genre path list (serialized).",
            "example": "Science Fiction & Fantasy > Science Fiction"
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 or storefront language code when known.",
            "example": "en"
          },
          "cover_url": {
            "type": [
              "string",
              "null"
            ],
            "description": "HTTPS URL for cover art when known.",
            "example": "https://images.example.com/covers/9780441478125.jpg"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known.",
            "example": 2499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields.",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display.",
            "example": "$24.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display.",
            "example": "$14.99"
          },
          "observed_at": {
            "type": [
              "string",
              "null"
            ],
            "format": "date-time",
            "description": "RFC 3339 time when this storefront snapshot was observed.",
            "example": "2026-10-29T10:12:00Z"
          },
          "created_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was inserted.",
            "example": "2026-10-29T10:12:00Z"
          },
          "updated_at": {
            "type": "string",
            "format": "date-time",
            "description": "RFC 3339 timestamp when the row was last modified.",
            "example": "2026-10-29T10:14:22Z"
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
      "description": "Blurb / synopsis text (may contain HTML).",
      "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
    },
    "subtitle": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional subtitle from bibliographic metadata.",
      "example": "A Hainish Novel"
    },
    "narrators": {
      "type": [
        "string",
        "null"
      ],
      "description": "Comma-separated or JSON narrator list when present.",
      "example": "George Guidall"
    },
    "series": {
      "type": [
        "string",
        "null"
      ],
      "description": "Series name when the title belongs to a series.",
      "example": "Hainish Cycle"
    },
    "series_index": {
      "type": [
        "string",
        "null"
      ],
      "description": "Position within the series (storefront string form).",
      "example": "4"
    },
    "publisher": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publisher name from metadata enrichment or the storefront.",
      "example": "Blackstone Publishing"
    },
    "length_minutes": {
      "type": [
        "integer",
        "null"
      ],
      "format": "int64",
      "description": "Runtime in whole minutes when the storefront reports it.",
      "example": 560
    },
    "published_at": {
      "type": [
        "string",
        "null"
      ],
      "description": "Publication date string from the storefront or enrichment.",
      "example": "2005-06-21"
    },
    "genres": {
      "type": [
        "string",
        "null"
      ],
      "description": "Merged genre / category list for display.",
      "example": "Science Fiction, Classics"
    },
    "language": {
      "type": [
        "string",
        "null"
      ],
      "description": "BCP-47 or storefront language code when known.",
      "example": "en"
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
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
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
            "description": "Content-source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title of the work or edition.",
            "example": "The Left Hand of Darkness"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront product or purchase URL when known.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Observed price in minor currency units, when known.",
            "example": 2499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields.",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted price string for display.",
            "example": "$24.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "List/MSRP price in minor units, when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted list price for display.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member/subscriber price in minor units, when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Storefront-formatted member price for display.",
            "example": "$14.99"
          }
        }
      },
      "description": "Purchase URLs and prices observed per storefront."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was inserted.",
      "example": "2026-10-29T10:12:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "RFC 3339 timestamp when the row was last modified.",
      "example": "2026-10-29T10:14:22Z"
    }
  }
}
```

Status `400`: Only open wishlist items can be removed.

```text
only open wishlist items can be removed
```

Schema:

A `text/plain` body: a string.

Status `401`: No signed-in caller.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: The request belongs to someone else. A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "client_sign_in_unavailable",
  "message": "This library does not accept client apps on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No such request.

```text
wishlist item not found: 0b7e4c1d-2a3f-4e5d-9c8b-7a6f5e4d3c2b
```

Schema:

A `text/plain` body: a string.

Status `500`: The request could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/request-queue`

Reads the shared open-request queue from its stored ranking.

The `discover_request_queue` job ranks the queue against household taste
when a taste event says it may have moved; this route joins that stored
ranking with the open requests and the live wish counts, so it reads no
books, listening history or embedder. When an entry has no stored ranking
yet, or the ranking is older than a day, the route queues a pass and
answers with what it has.

**Access:** The operator, or any library session, including a client sign-in's token (refused with 403 `transport_insecure` off loopback without TLS, or `client_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clientToken`

### Responses

- `200` — Open requests, ranked. Body: array of [`RankedQueueEntry`](schemas.md#rankedqueueentry) (`application/json`)
  - `work_key` *string*, **required** — Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).
  - `work_id` *string*, **required** — The work every wish of this entry is for. Wishes for one work are one entry even when their `work_key`s differ, so a client matches its own wishes on this id first and on `work_key` otherwise.
  - `title` *string*, **required** — Display title as shown on the storefront or library card.
  - `authors` *string or null* — Comma-separated author names when the storefront provides them.
  - `asin` *string or null* — Audible / Amazon ASIN when this edition is sold on Audible.
  - `isbn` *string or null* — Canonical ISBN-13 (or ISBN-10 normalized) when published.
  - `cover_url` *string or null* — HTTPS URL for cover art when the catalog exposes one.
  - `cover_path` *string or null* — Same-origin path of this cover's processed images, set by the daemon when `cover_url` is an `https` URL. Append `/128` (thumbnail), `/360` (card) or `/full`, so a client can show covers without loading `cover_url` from another site.
  - `wish_count` *integer (int64)*, **required** — Number of distinct users who wishlisted this title.
  - `wishers` *array of [`QueueWisher`](schemas.md#queuewisher)* — Distinct people who have this title on an open wishlist.
  - `sample_uuids` *array of string*, **required** — Sample Bookclerk user UUIDs who requested this title (privacy-capped).
  - `first_requested_at` *string (date-time)*, **required** — UTC timestamp of the earliest wishlist request for this work.
  - `last_requested_at` *string (date-time)*, **required** — UTC timestamp of the latest wishlist request for this work.
  - `score` *number (double)*, **required** — Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`]).
  - `taste_score` *number (double)*, **required** — Local taste / embedding component before the wish-count boost.
  - `reasons` *array of string*, **required** — Short human-readable explanations for why this item was ranked here.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The queue could not be read. Body: string (`text/plain`)
- `503` — The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
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
    "work_key": "isbn:9780441478125",
    "work_id": "0b7e2c1d-5f3a-4e6b-9c8d-1a2b3c4d5e6f",
    "title": "The Left Hand of Darkness",
    "authors": "Ursula K. Le Guin",
    "asin": "B002V0QK4C",
    "isbn": "9780441478125",
    "cover_url": "https://images.example.com/covers/9780441478125.jpg",
    "cover_path": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA",
    "wish_count": 3,
    "wishers": [
      {
        "user_id": 7,
        "identity_id": 7,
        "display_name": "Dana Reader",
        "login_name": "dana",
        "operator": false,
        "has_avatar": true,
        "avatar_source": "upload",
        "gravatar_hash": "84d2a1a1d9ba6a4c2f4b9d0e7a7c6b5f3e2d1c0b9a8f7e6d5c4b3a2918f7e6d5",
        "picture_url": "https://accounts.example.com/u/dana/picture.png",
        "picture_path": "/api/images/aHR0cHM6Ly9hY2NvdW50cy5leGFtcGxlLmNvbS91L2RhbmEvcGljdHVyZS5wbmc.N14rqh66Bok9-eCqJKBNoQ"
      }
    ],
    "sample_uuids": [
      "0b7e2c1d-5f3a-4e6b-9c8d-1a2b3c4d5e6f"
    ],
    "first_requested_at": "2026-09-02T08:15:00Z",
    "last_requested_at": "2026-10-29T10:12:00Z",
    "score": 120.61,
    "taste_score": 0.61,
    "reasons": [
      "wishlisted by 3 people",
      "matches liked author (Ursula K. Le Guin)"
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
      "work_id",
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
        "description": "Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).",
        "example": "isbn:9780441478125"
      },
      "work_id": {
        "type": "string",
        "description": "The work every wish of this entry is for. Wishes for one work are\none entry even when their `work_key`s differ, so a client matches\nits own wishes on this id first and on `work_key` otherwise.",
        "example": "0b7e2c1d-5f3a-4e6b-9c8d-1a2b3c4d5e6f"
      },
      "title": {
        "type": "string",
        "description": "Display title as shown on the storefront or library card.",
        "example": "The Left Hand of Darkness"
      },
      "authors": {
        "type": [
          "string",
          "null"
        ],
        "description": "Comma-separated author names when the storefront provides them.",
        "example": "Ursula K. Le Guin"
      },
      "asin": {
        "type": [
          "string",
          "null"
        ],
        "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
        "example": "B002V0QK4C"
      },
      "isbn": {
        "type": [
          "string",
          "null"
        ],
        "description": "Canonical ISBN-13 (or ISBN-10 normalized) when published.",
        "example": "9780441478125"
      },
      "cover_url": {
        "type": [
          "string",
          "null"
        ],
        "description": "HTTPS URL for cover art when the catalog exposes one.",
        "example": "https://images.example.com/covers/9780441478125.jpg"
      },
      "cover_path": {
        "type": [
          "string",
          "null"
        ],
        "description": "Same-origin path of this cover's processed images, set by the daemon\nwhen `cover_url` is an `https` URL. Append `/128`\n(thumbnail), `/360` (card) or `/full`, so a client can show covers\nwithout loading `cover_url` from another site.",
        "example": "/api/covers/aHR0cHM6Ly9pbWFnZXMuZXhhbXBsZS5jb20vYy5qcGc.q2fW1n4bJm0Rg9xV3kTzPA"
      },
      "wish_count": {
        "type": "integer",
        "format": "int64",
        "description": "Number of distinct users who wishlisted this title.",
        "example": 3
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
              "description": "First-party `users.id` when the portal identity is linked.",
              "example": 7
            },
            "identity_id": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Portal identity that created the wish (dedupe for unlinked identities).",
              "example": 7
            },
            "display_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display name, identity label, or `\"Operator\"`.",
              "example": "Dana Reader"
            },
            "login_name": {
              "type": [
                "string",
                "null"
              ],
              "description": "Local login name when the first-party user has one.",
              "example": "dana"
            },
            "operator": {
              "type": "boolean",
              "description": "True when the wish came from the operator token (no portal user).",
              "example": false
            },
            "has_avatar": {
              "type": "boolean",
              "description": "True when an uploaded avatar file exists (filled by the daemon).",
              "example": true
            },
            "avatar_source": {
              "type": [
                "string",
                "null"
              ],
              "description": "Explicit picture choice when the user has set one.",
              "example": "upload"
            },
            "gravatar_hash": {
              "type": [
                "string",
                "null"
              ],
              "description": "SHA-256 hex of the contact email for Gravatar.",
              "example": "84d2a1a1d9ba6a4c2f4b9d0e7a7c6b5f3e2d1c0b9a8f7e6d5c4b3a2918f7e6d5"
            },
            "picture_url": {
              "type": [
                "string",
                "null"
              ],
              "description": "HTTPS picture from the wishing identity, when any.",
              "example": "https://accounts.example.com/u/dana/picture.png"
            },
            "picture_path": {
              "type": [
                "string",
                "null"
              ],
              "description": "Same-origin 128 px copy of `picture_url` for the web app to show\n(filled by the daemon; never stored).",
              "example": "/api/images/aHR0cHM6Ly9hY2NvdW50cy5leGFtcGxlLmNvbS91L2RhbmEvcGljdHVyZS5wbmc.N14rqh66Bok9-eCqJKBNoQ"
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
        "description": "Sample Bookclerk user UUIDs who requested this title (privacy-capped).",
        "example": [
          "0b7e2c1d-5f3a-4e6b-9c8d-1a2b3c4d5e6f"
        ]
      },
      "first_requested_at": {
        "type": "string",
        "format": "date-time",
        "description": "UTC timestamp of the earliest wishlist request for this work.",
        "example": "2026-09-02T08:15:00Z"
      },
      "last_requested_at": {
        "type": "string",
        "format": "date-time",
        "description": "UTC timestamp of the latest wishlist request for this work.",
        "example": "2026-10-29T10:12:00Z"
      },
      "score": {
        "type": "number",
        "format": "double",
        "description": "Final rank score (`taste_score + wish_count * `[`WISH_COUNT_WEIGHT`]).",
        "example": 120.61
      },
      "taste_score": {
        "type": "number",
        "format": "double",
        "description": "Local taste / embedding component before the wish-count boost.",
        "example": 0.61
      },
      "reasons": {
        "type": "array",
        "items": {
          "type": "string"
        },
        "description": "Short human-readable explanations for why this item was ranked here.",
        "example": [
          "wishlisted by 3 people",
          "matches liked author (Ursula K. Le Guin)"
        ]
      }
    }
  }
}
```

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`.

```json
{
  "error": "client_sign_in_unavailable",
  "message": "This library does not accept client apps on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `500`: The queue could not be read.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```json
{
  "error": "session_check_unavailable",
  "message": "Your sign-in could not be checked yet. Try again in a moment.",
  "status": 503
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/branch-requests`

Asks a branch for a book it listed.

The work must currently be available through a branch this library
trusts (its `available_through` in Discover and search). The branch
takes the request automatically for anything it listed; the book
arrives with the branch's catalog sync and is then held through that
branch. Asking again while a request for the same work and branch
waits returns that request.

**Access:** The operator, or any library session, including a client sign-in's token (refused with 403 `transport_insecure` off loopback without TLS, or `client_sign_in_unavailable` where remote installs are not served).

**Credentials:** `operatorToken` or `operatorSession` or `portalSession` or `clientToken`

### Request body

`application/json`: [`BranchRequestBody`](schemas.md#branchrequestbody)

- `work_id` *string*, **required** — This library's work id, as Discover and search name it.
- `node_id` *string or null* — The branch to ask, from the work's `available_through`; absent or `null` asks the first branch that lists the work.

### Responses

- `200` — The request was queued for the branch. Body: [`BranchRequestQueued`](schemas.md#branchrequestqueued) (`application/json`)
  - `job_id` *string*, **required** — The request's job (kind `branch_request`), as `/api/jobs` lists it.
  - `node_id` *string*, **required** — The branch the request went to.
  - `already_queued` *boolean*, **required** — `true` when a request for this work to this branch was already waiting or being taken; `job_id` is that request.
- `400` — The work or node id is malformed. As `text/plain`, when the body is not syntactically valid JSON. Body: string (`text/plain`)
- `401` — The caller is not signed in. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `404` — No branch this library trusts (or not the named one) currently lists the work. Body: string (`text/plain`)
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `500` — The library could not be read or written. Body: string (`text/plain`)
- `503` — The job queue is full; try again later. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/branch-requests' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/branch-requests' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/branch-requests",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/branch-requests", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/branch-requests")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"work_id": "0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The request was queued for the branch.

```json
{
  "job_id": "branch_request-5e0f2a1c-7b3d-4c6e-8f9a-1b2c3d4e5f6a",
  "node_id": "9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e",
  "already_queued": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "`POST /api/branch-requests` answer.",
  "required": [
    "job_id",
    "node_id",
    "already_queued"
  ],
  "properties": {
    "job_id": {
      "type": "string",
      "description": "The request's job (kind `branch_request`), as `/api/jobs` lists it.",
      "example": "branch_request-5e0f2a1c-7b3d-4c6e-8f9a-1b2c3d4e5f6a"
    },
    "node_id": {
      "type": "string",
      "description": "The branch the request went to.",
      "example": "9b1c7d2e-4f3a-4b6c-8d1e-2f3a4b5c6d7e"
    },
    "already_queued": {
      "type": "boolean",
      "description": "`true` when a request for this work to this branch was already\nwaiting or being taken; `job_id` is that request.",
      "example": false
    }
  }
}
```

Status `400`: The work or node id is malformed. As `text/plain`, when the body is not syntactically valid JSON.

```text
Failed to parse the request body as JSON: expected value at line 1 column 1
```

Schema:

A `text/plain` body: a string.

Status `401`: The caller is not signed in.

```json
{
  "error": "unauthorized",
  "message": "Operator authentication is required. Sign in with your operator token.",
  "status": 401
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
      "description": "Stable machine slug.",
      "examples": [
        "not_found"
      ]
    },
    "message": {
      "type": "string",
      "description": "Explanation for people.",
      "examples": [
        "That page or resource was not found."
      ]
    },
    "status": {
      "type": "integer",
      "description": "The HTTP status, repeated.",
      "examples": [
        404
      ]
    }
  }
}
```

Status `403`: A signed-in user who must enroll a second factor first gets `mfa_enrollment_required`. A library card sent over plain HTTP from another machine gets `transport_insecure`. A library card sent where the node API is not mounted gets `client_sign_in_unavailable`. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "client_sign_in_unavailable",
  "message": "This library does not accept client apps on this address.",
  "status": 403
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object (`error`, `message`, numeric `status`) for API clients.",
  "required": [
    "error",
    "message",
    "status"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Machine slug (same as [`ErrorBody::code`]).",
      "example": "not_found"
    },
    "message": {
      "type": "string",
      "description": "Operator-facing explanation.",
      "example": "That page or resource was not found."
    },
    "status": {
      "type": "integer",
      "format": "int32",
      "description": "Numeric HTTP status (duplicates the response status for clients).",
      "example": 404,
      "minimum": 0
    }
  }
}
```

Status `404`: No branch this library trusts (or not the named one) currently lists the work.

```text
book not found: work 0b8f6a52-3c1d-4e2f-9a7b-6c5d4e3f2a1b through a branch
```

Schema:

A `text/plain` body: a string.

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `work_id` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `500`: The library could not be read or written.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: The job queue is full; try again later. The sign-in could not be checked because the library did not answer in time. The response carries `Retry-After`; ask again rather than treating the caller as signed out.

```text
the job queue is full
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
