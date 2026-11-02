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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The same job is already pending or running, or no node can run it. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `429` — The job queue is full. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `500` — The job could not be queued. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `503` — Job admission is paused. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
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
  "job_id": "listen_sync-2b1a0f9e-8d7c-4b6a-9f5e-4d3c2b1a0f9e",
  "message": "listen_sync job listen_sync-2b1a0f9e-8d7c-4b6a-9f5e-4d3c2b1a0f9e accepted",
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `400`: No listening-capable integration is enabled.

```text
no listening-capable integrations are enabled
```

Schema:

A `text/plain` body: a string.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `409`: The same job is already pending or running, or no node can run it.

```json
{
  "error": "conflict",
  "job_id": "listen_sync-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f",
  "message": "listen_sync already pending or running",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `429`: The job queue is full.

```json
{
  "error": "queue_full",
  "job_id": "",
  "message": "job queue is full",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `500`: The job could not be queued.

```json
{
  "error": "internal_error",
  "job_id": "",
  "message": "ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
    }
  }
}
```

Status `503`: Job admission is paused.

```json
{
  "error": "admission_paused",
  "job_id": "",
  "message": "job admission paused (database reload in progress)",
  "ok": false
}
```

Schema:

```json
{
  "type": "object",
  "description": "Generic mutation result with success flag, message, and optional job id.",
  "required": [
    "ok",
    "message",
    "job_id"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "True when the action was accepted or completed.",
      "example": true
    },
    "message": {
      "type": "string",
      "description": "Operator-facing status text.",
      "example": "scan queued"
    },
    "job_id": {
      "type": "string",
      "description": "Queued job id, or empty when the action is synchronous.",
      "example": "scan-7d0c1e2f-3a4b-4c5d-8e6f-9a0b1c2d3e4f"
    },
    "error": {
      "type": [
        "string",
        "null"
      ],
      "description": "Machine slug when admission failed (`conflict`, `queue_full`).",
      "example": "queue_full"
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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The lookup failed. Body: string (`text/plain`)
- `503` — Discover is shutting down; try again. Body: string (`text/plain`)
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
      "id": "author:ursula-k-le-guin",
      "title": "More from Ursula K. Le Guin",
      "subtitle": "Because you’ve liked their work",
      "items": [
        {
          "title": "The Left Hand of Darkness",
          "score": 120.87,
          "reasons": [
            "wishlisted by 3 people",
            "matches liked author (Ursula K. Le Guin)"
          ],
          "purchase_hints": [],
          "from_request": true
        }
      ]
    }
  ],
  "shelf_kinds": [
    {
      "id": "finish_series",
      "label": "Finish these series"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Full Discover page payload: ordered shelves of personalized candidates.",
  "required": [
    "shelves"
  ],
  "properties": {
    "shelves": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One horizontal Discover row (series to finish, more by author, …).",
        "required": [
          "id",
          "title",
          "items"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable identifier for this item.",
            "example": "author:ursula-k-le-guin"
          },
          "title": {
            "type": "string",
            "description": "Display title as shown on the storefront or library card.",
            "example": "More from Ursula K. Le Guin"
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle when the catalog distinguishes it from the title.",
            "example": "Because you’ve liked their work"
          },
          "items": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One ranked recommendation (typically an unowned storefront title).",
              "required": [
                "title",
                "score",
                "reasons",
                "purchase_hints",
                "from_request"
              ],
              "properties": {
                "work_id": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Library `works.id` UUID when the recommendation is tied to an owned work.",
                  "example": "5b7e2c1d-8a4f-4e3b-9c6d-2f1a0b9e8d7c"
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
                "narrators": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Comma-separated narrator names when known.",
                  "example": "George Guidall"
                },
                "series": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Series name when the title belongs to a named series.",
                  "example": "Hainish Cycle"
                },
                "series_index": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Position within the series (e.g. `1`, `1.5`) when known.",
                  "example": "4"
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
                "score": {
                  "type": "number",
                  "format": "double",
                  "description": "Ranking score (higher is better); units are algorithm-specific.",
                  "example": 120.87
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
                },
                "purchase_hints": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "A purchase / catalog availability hint (optionally priced at view time).",
                    "required": [
                      "source",
                      "product_id"
                    ],
                    "properties": {
                      "source": {
                        "type": "string",
                        "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                        "example": "libro"
                      },
                      "product_id": {
                        "type": "string",
                        "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                        "example": "9780441478125"
                      },
                      "title": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display title as shown on the storefront or library card.",
                        "example": "The Left Hand of Darkness"
                      },
                      "url": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Absolute HTTPS URL.",
                        "example": "https://libro.fm/audiobooks/9780441478125"
                      },
                      "price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                        "example": 1499
                      },
                      "currency": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                        "example": "USD"
                      },
                      "price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Display string from the store (`$2.99`, `FREE`, …).",
                        "example": "$14.99"
                      },
                      "list_price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Non-member / list price in integer cents when known.",
                        "example": 2999
                      },
                      "list_price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Human-readable list price from the storefront.",
                        "example": "$29.99"
                      },
                      "member_price_cents": {
                        "type": [
                          "integer",
                          "null"
                        ],
                        "format": "int64",
                        "description": "Member / credit price in integer cents when known.",
                        "example": 1499
                      },
                      "member_price_label": {
                        "type": [
                          "string",
                          "null"
                        ],
                        "description": "Human-readable member price from the storefront.",
                        "example": "$14.99"
                      }
                    }
                  },
                  "description": "Buy / open-in-store links with optional member vs list pricing."
                },
                "from_request": {
                  "type": "boolean",
                  "description": "True when sourced from an open title request rather than storefront discovery.",
                  "example": true
                },
                "request_uuid": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Wishlist / title-request UUID when this card came from a request.",
                  "example": "7c1e5a2b-3d4f-4e6a-9b8c-0d1e2f3a4b5c"
                },
                "candidate_source": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Storefront that proposed this title (`audible`, `libro`, …).",
                  "example": "libro"
                },
                "candidate_product_id": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Storefront product id of the edition that produced this card.",
                  "example": "9780441478125"
                },
                "store_editions": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One storefront edition of a work (for multi-store purchase links).",
                    "required": [
                      "source",
                      "product_id"
                    ],
                    "properties": {
                      "source": {
                        "type": "string",
                        "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                        "example": "libro"
                      },
                      "product_id": {
                        "type": "string",
                        "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                        "example": "9780441478125"
                      }
                    }
                  },
                  "description": "All known storefront editions (for multi-store purchase links)."
                },
                "seed_categories": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Categories/subjects copied from the taste seed that produced this hit.",
                  "example": "Science Fiction & Fantasy > Science Fiction"
                },
                "categories": {
                  "type": "array",
                  "items": {
                    "type": "string"
                  },
                  "description": "Stable shelf-kind tags for prefs (`finish_series`, `author`, `chirp_deals`, …).\nDynamic shelf titles stay in `build_discover_feed`; membership uses these.",
                  "example": [
                    "finish_series",
                    "author"
                  ]
                },
                "work_key": {
                  "type": "string",
                  "description": "Stable bibliographic key for wishlist / merge (`isbn:` / `asin:` / `soft:`…).",
                  "example": "isbn:9780441478125"
                },
                "cover_url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Cover art URL when known from a storefront catalog hit or wishlist row.",
                  "example": "https://images.example.com/covers/9780441478125.jpg"
                },
                "subtitle": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Optional bibliographic extras from storefront catalog / Audnexus.",
                  "example": "A Hainish Novel"
                },
                "description": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
                  "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
                },
                "publisher": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publisher imprint as reported by the catalog.",
                  "example": "Blackstone Publishing"
                },
                "length_minutes": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Audiobook runtime in whole minutes when known.",
                  "example": 560
                },
                "published_at": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Publication date string from the catalog (ISO or storefront format).",
                  "example": "2005-06-21"
                },
                "genres": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Candidate genres (not taste-seed `seed_categories`).",
                  "example": "Science Fiction, Classics"
                },
                "language": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
                  "example": "en"
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
                      }
                    }
                  },
                  "description": "Distinct people who have this title on an open wishlist."
                },
                "wish_count": {
                  "type": "integer",
                  "format": "int64",
                  "description": "Number of distinct users who wishlisted this title (may exceed `wishers.len()`).",
                  "example": 3
                }
              }
            },
            "description": "Ordered collection of child entries for this response."
          }
        }
      },
      "description": "Ordered horizontal shelves that make up the Discover page."
    },
    "shelf_kinds": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A shelf kind the operator can ignore (all offered by default).",
        "required": [
          "id",
          "label"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Stable kind id (`finish_series`, `author`, `genre`, `from_store`, …).",
            "example": "finish_series"
          },
          "label": {
            "type": "string",
            "description": "Operator-visible label for this account or item.",
            "example": "Finish these series"
          }
        }
      },
      "description": "Catalog of shelf kinds for ignore prefs (empty `disabled_shelves` = all on)."
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
Failed to deserialize query string: invalid digit found in string
```

Schema:

A `text/plain` body: a string.

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

Status `500`: The lookup failed.

```text
ORM / database plugin error: Connection Error: pool timed out while waiting for an open connection
```

Schema:

A `text/plain` body: a string.

Status `503`: Discover is shutting down; try again.

```text
discover gate closed
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/purchase-hints' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"title": "The Left Hand of Darkness"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/purchase-hints' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"title": "The Left Hand of Darkness"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/purchase-hints",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"title": "The Left Hand of Darkness"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/purchase-hints", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"title": "The Left Hand of Darkness"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/purchase-hints")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"title": "The Left Hand of Darkness"}))
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
      "source": "libro",
      "product_id": "9780441478125",
      "title": "The Left Hand of Darkness",
      "url": "https://libro.fm/audiobooks/9780441478125",
      "price_cents": 1499,
      "currency": "USD",
      "price_label": "$14.99",
      "list_price_cents": 2999,
      "list_price_label": "$29.99",
      "member_price_cents": 1499,
      "member_price_label": "$14.99"
    }
  ],
  "best": {
    "source": "libro",
    "product_id": "9780441478125",
    "title": "The Left Hand of Darkness",
    "url": "https://libro.fm/audiobooks/9780441478125",
    "price_cents": 1499,
    "currency": "USD",
    "price_label": "$14.99",
    "list_price_cents": 2999,
    "list_price_label": "$29.99",
    "member_price_cents": 1499,
    "member_price_label": "$14.99"
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Priced catalog matches for one title, sorted best-first.",
  "required": [
    "hints"
  ],
  "properties": {
    "hints": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "A purchase / catalog availability hint (optionally priced at view time).",
        "required": [
          "source",
          "product_id"
        ],
        "properties": {
          "source": {
            "type": "string",
            "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
            "example": "libro"
          },
          "product_id": {
            "type": "string",
            "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
            "example": "9780441478125"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title as shown on the storefront or library card.",
            "example": "The Left Hand of Darkness"
          },
          "url": {
            "type": [
              "string",
              "null"
            ],
            "description": "Absolute HTTPS URL.",
            "example": "https://libro.fm/audiobooks/9780441478125"
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
            "example": 1499
          },
          "currency": {
            "type": [
              "string",
              "null"
            ],
            "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
            "example": "USD"
          },
          "price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display string from the store (`$2.99`, `FREE`, …).",
            "example": "$14.99"
          },
          "list_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Non-member / list price in integer cents when known.",
            "example": 2999
          },
          "list_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Human-readable list price from the storefront.",
            "example": "$29.99"
          },
          "member_price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Member / credit price in integer cents when known.",
            "example": 1499
          },
          "member_price_label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Human-readable member price from the storefront.",
            "example": "$14.99"
          }
        }
      },
      "description": "Resolved purchase hints for this query."
    },
    "best": {
      "oneOf": [
        {
          "type": "object",
          "description": "Lowest-priced hint (or first catalog hit when no prices resolved).",
          "required": [
            "source",
            "product_id"
          ],
          "properties": {
            "source": {
              "type": "string",
              "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
              "example": "libro"
            },
            "product_id": {
              "type": "string",
              "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
              "example": "9780441478125"
            },
            "title": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display title as shown on the storefront or library card.",
              "example": "The Left Hand of Darkness"
            },
            "url": {
              "type": [
                "string",
                "null"
              ],
              "description": "Absolute HTTPS URL.",
              "example": "https://libro.fm/audiobooks/9780441478125"
            },
            "price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
              "example": 1499
            },
            "currency": {
              "type": [
                "string",
                "null"
              ],
              "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
              "example": "USD"
            },
            "price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Display string from the store (`$2.99`, `FREE`, …).",
              "example": "$14.99"
            },
            "list_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Non-member / list price in integer cents when known.",
              "example": 2999
            },
            "list_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Human-readable list price from the storefront.",
              "example": "$29.99"
            },
            "member_price_cents": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int64",
              "description": "Member / credit price in integer cents when known.",
              "example": 1499
            },
            "member_price_label": {
              "type": [
                "string",
                "null"
              ],
              "description": "Human-readable member price from the storefront.",
              "example": "$14.99"
            }
          }
        },
        {
          "type": "null"
        }
      ]
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
title, asin, isbn, or candidate_product_id is required
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — A lookup task was cancelled before it answered. Body: string (`text/plain`)
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
          "source": "libro",
          "product_id": "9780441478125"
        }
      ],
      "best": {
        "source": "libro",
        "product_id": "9780441478125",
        "title": "The Left Hand of Darkness",
        "url": "https://libro.fm/audiobooks/9780441478125",
        "price_cents": 1499,
        "currency": "USD",
        "price_label": "$14.99",
        "list_price_cents": 2999,
        "list_price_label": "$29.99",
        "member_price_cents": 1499,
        "member_price_label": "$14.99"
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Parallel purchase-hint results aligned with the request queries.",
  "required": [
    "results"
  ],
  "properties": {
    "results": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "Priced catalog matches for one title, sorted best-first.",
        "required": [
          "hints"
        ],
        "properties": {
          "hints": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "A purchase / catalog availability hint (optionally priced at view time).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                },
                "title": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display title as shown on the storefront or library card.",
                  "example": "The Left Hand of Darkness"
                },
                "url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Absolute HTTPS URL.",
                  "example": "https://libro.fm/audiobooks/9780441478125"
                },
                "price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                  "example": 1499
                },
                "currency": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                  "example": "USD"
                },
                "price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display string from the store (`$2.99`, `FREE`, …).",
                  "example": "$14.99"
                },
                "list_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Non-member / list price in integer cents when known.",
                  "example": 2999
                },
                "list_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable list price from the storefront.",
                  "example": "$29.99"
                },
                "member_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Member / credit price in integer cents when known.",
                  "example": 1499
                },
                "member_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable member price from the storefront.",
                  "example": "$14.99"
                }
              }
            },
            "description": "Resolved purchase hints for this query."
          },
          "best": {
            "oneOf": [
              {
                "type": "object",
                "description": "Lowest-priced hint (or first catalog hit when no prices resolved).",
                "required": [
                  "source",
                  "product_id"
                ],
                "properties": {
                  "source": {
                    "type": "string",
                    "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                    "example": "libro"
                  },
                  "product_id": {
                    "type": "string",
                    "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                    "example": "9780441478125"
                  },
                  "title": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display title as shown on the storefront or library card.",
                    "example": "The Left Hand of Darkness"
                  },
                  "url": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Absolute HTTPS URL.",
                    "example": "https://libro.fm/audiobooks/9780441478125"
                  },
                  "price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                    "example": 1499
                  },
                  "currency": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                    "example": "USD"
                  },
                  "price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Display string from the store (`$2.99`, `FREE`, …).",
                    "example": "$14.99"
                  },
                  "list_price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Non-member / list price in integer cents when known.",
                    "example": 2999
                  },
                  "list_price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Human-readable list price from the storefront.",
                    "example": "$29.99"
                  },
                  "member_price_cents": {
                    "type": [
                      "integer",
                      "null"
                    ],
                    "format": "int64",
                    "description": "Member / credit price in integer cents when known.",
                    "example": 1499
                  },
                  "member_price_label": {
                    "type": [
                      "string",
                      "null"
                    ],
                    "description": "Human-readable member price from the storefront.",
                    "example": "$14.99"
                  }
                }
              },
              {
                "type": "null"
              }
            ]
          }
        }
      },
      "description": "One purchase-hint response per query, in request order."
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
at most 24 purchase-hint queries per batch
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `500`: A lookup task was cancelled before it answered.

```text
purchase-hints batch task cancelled
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The stores did not answer within the search budget. Body: string (`text/plain`)

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
      "work_key": "isbn:9780441478125",
      "title": "The Left Hand of Darkness",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "asin": "B002V0QK4C",
      "isbn": "9780441478125",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "store_editions": [
        {
          "source": "libro",
          "product_id": "9780441478125"
        }
      ],
      "sources": [
        "libro",
        "chirp"
      ],
      "subtitle": "A Hainish Novel",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "genres": "Science Fiction, Classics",
      "language": "en",
      "is_abridged": false,
      "rating_overall": 4.5,
      "rating_count": 12840,
      "price_cents": 2499,
      "purchase_hints": [
        {
          "source": "libro",
          "product_id": "9780441478125"
        }
      ]
    }
  ],
  "page_size": 20,
  "has_more": true,
  "next_cursor": "s1:4f1c9e2a-7b3d-4c8e-9a6f-0d2b1e3c5a7f",
  "sort": "relevance",
  "sort_dir": "desc"
}
```

Schema:

```json
{
  "type": "object",
  "description": "Paged catalog-search response.",
  "required": [
    "items",
    "page_size",
    "has_more",
    "sort"
  ],
  "properties": {
    "items": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One autocomplete / results-page hit (possibly spanning multiple storefronts).",
        "required": [
          "work_key",
          "title",
          "store_editions",
          "sources"
        ],
        "properties": {
          "work_key": {
            "type": "string",
            "description": "Stable map key for this work (`isbn:…`, `asin:…`, or soft title+author).",
            "example": "isbn:9780441478125"
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
          "narrators": {
            "type": [
              "string",
              "null"
            ],
            "description": "Comma-separated narrator names when known.",
            "example": "George Guidall"
          },
          "series": {
            "type": [
              "string",
              "null"
            ],
            "description": "Series name when the title belongs to a named series.",
            "example": "Hainish Cycle"
          },
          "series_index": {
            "type": [
              "string",
              "null"
            ],
            "description": "Position within the series (e.g. `1`, `1.5`) when known.",
            "example": "4"
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
          "store_editions": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One storefront edition of a work (for multi-store purchase links).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                }
              }
            },
            "description": "Per-storefront edition ids that collapsed into this work card."
          },
          "sources": {
            "type": "array",
            "items": {
              "type": "string"
            },
            "description": "Storefronts that matched (deduped source ids).",
            "example": [
              "libro",
              "chirp"
            ]
          },
          "subtitle": {
            "type": [
              "string",
              "null"
            ],
            "description": "Optional subtitle when the catalog distinguishes it from the title.",
            "example": "A Hainish Novel"
          },
          "description": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
            "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
          },
          "publisher": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publisher imprint as reported by the catalog.",
            "example": "Blackstone Publishing"
          },
          "length_minutes": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Audiobook runtime in whole minutes when known.",
            "example": 560
          },
          "published_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "Publication date string from the catalog (ISO or storefront format).",
            "example": "2005-06-21"
          },
          "genres": {
            "type": [
              "string",
              "null"
            ],
            "description": "Genre labels from the catalog (comma-separated when multiple).",
            "example": "Science Fiction, Classics"
          },
          "language": {
            "type": [
              "string",
              "null"
            ],
            "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
            "example": "en"
          },
          "is_abridged": {
            "type": [
              "boolean",
              "null"
            ],
            "description": "When `Some(true)`, the edition is marked abridged by the storefront.",
            "example": false
          },
          "rating_overall": {
            "type": [
              "number",
              "null"
            ],
            "format": "double",
            "description": "Aggregate listener rating on the storefront's scale when known.",
            "example": 4.5
          },
          "rating_count": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Number of ratings backing [`Self::rating_overall`] when known.",
            "example": 12840
          },
          "price_cents": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Catalog list/deal price in cents when the storefront provided it.",
            "example": 2499
          },
          "purchase_hints": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "A purchase / catalog availability hint (optionally priced at view time).",
              "required": [
                "source",
                "product_id"
              ],
              "properties": {
                "source": {
                  "type": "string",
                  "description": "Canonical storefront / plugin id (`audible`, `libro`, `chirp`, …).",
                  "example": "libro"
                },
                "product_id": {
                  "type": "string",
                  "description": "Storefront-native product id (ASIN, ISBN, UUID, …).",
                  "example": "9780441478125"
                },
                "title": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display title as shown on the storefront or library card.",
                  "example": "The Left Hand of Darkness"
                },
                "url": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Absolute HTTPS URL.",
                  "example": "https://libro.fm/audiobooks/9780441478125"
                },
                "price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Primary / best known sell price in minor units (prefer member when dual).\n`0` = free.",
                  "example": 1499
                },
                "currency": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "ISO 4217 currency code for price fields (`USD`, `EUR`, …).",
                  "example": "USD"
                },
                "price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Display string from the store (`$2.99`, `FREE`, …).",
                  "example": "$14.99"
                },
                "list_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Non-member / list price in integer cents when known.",
                  "example": 2999
                },
                "list_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable list price from the storefront.",
                  "example": "$29.99"
                },
                "member_price_cents": {
                  "type": [
                    "integer",
                    "null"
                  ],
                  "format": "int64",
                  "description": "Member / credit price in integer cents when known.",
                  "example": 1499
                },
                "member_price_label": {
                  "type": [
                    "string",
                    "null"
                  ],
                  "description": "Human-readable member price from the storefront.",
                  "example": "$14.99"
                }
              }
            },
            "description": "Buy / open-in-store links with optional member vs list pricing."
          }
        }
      },
      "description": "Ordered collection of child entries for this response."
    },
    "page_size": {
      "type": "integer",
      "description": "Maximum number of items returned in this page.",
      "example": 20,
      "minimum": 0
    },
    "has_more": {
      "type": "boolean",
      "description": "When true, another page is available via [`Self::next_cursor`].",
      "example": true
    },
    "next_cursor": {
      "type": [
        "string",
        "null"
      ],
      "description": "Opaque cursor for the next page; omit or `None` when exhausted.",
      "example": "s1:4f1c9e2a-7b3d-4c8e-9a6f-0d2b1e3c5a7f"
    },
    "sort": {
      "type": "string",
      "description": "Sort field applied to the merged result set.",
      "example": "relevance"
    },
    "sort_dir": {
      "type": "string",
      "description": "Ascending or descending order for [`Self::sort`].",
      "example": "desc"
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
catalog query must be 256 bytes or fewer
```

Schema:

A `text/plain` body: a string.

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

Status `504`: The stores did not answer within the search budget.

```text
catalog search timed out
```

Schema:

A `text/plain` body: a string.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
  "asin": "B002V0QK4C",
  "title": "The Left Hand of Darkness",
  "subtitle": "A Hainish Novel",
  "authors": "Ursula K. Le Guin",
  "narrators": "George Guidall",
  "series": "Hainish Cycle",
  "series_index": "4",
  "isbn": "9780441478125",
  "cover_url": "https://images.example.com/covers/9780441478125.jpg",
  "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
  "publisher": "Blackstone Publishing",
  "length_minutes": 560,
  "published_at": "2005-06-21",
  "categories": "Science Fiction & Fantasy; Science Fiction",
  "language": "en",
  "is_abridged": false,
  "rating_overall": 4.5,
  "rating_performance": 4.7,
  "rating_story": 4.3,
  "rating_count": 12840,
  "review_count": 1932,
  "reviews": [
    {
      "id": "R3K2M9P1Q7X4Z",
      "title": "Still the best",
      "body": "Guidall's narration carries the quiet of Winter; I listened twice.",
      "author_name": "Dana",
      "overall_rating": 5,
      "performance_rating": 5,
      "story_rating": 4,
      "submitted_at": "2026-02-11"
    }
  ]
}
```

Schema:

```json
{
  "oneOf": [
    {
      "type": "object",
      "description": "Bibliographic fields suitable for an Audible-style detail panel.",
      "properties": {
        "asin": {
          "type": [
            "string",
            "null"
          ],
          "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
          "example": "B002V0QK4C"
        },
        "title": {
          "type": [
            "string",
            "null"
          ],
          "description": "Display title as shown on the storefront or library card.",
          "example": "The Left Hand of Darkness"
        },
        "subtitle": {
          "type": [
            "string",
            "null"
          ],
          "description": "Optional subtitle when the catalog distinguishes it from the title.",
          "example": "A Hainish Novel"
        },
        "authors": {
          "type": [
            "string",
            "null"
          ],
          "description": "Comma-separated author names when the storefront provides them.",
          "example": "Ursula K. Le Guin"
        },
        "narrators": {
          "type": [
            "string",
            "null"
          ],
          "description": "Comma-separated narrator names when known.",
          "example": "George Guidall"
        },
        "series": {
          "type": [
            "string",
            "null"
          ],
          "description": "Series name when the title belongs to a named series.",
          "example": "Hainish Cycle"
        },
        "series_index": {
          "type": [
            "string",
            "null"
          ],
          "description": "Position within the series (e.g. `1`, `1.5`) when known.",
          "example": "4"
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
        "description": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
          "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
        },
        "publisher": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publisher imprint as reported by the catalog.",
          "example": "Blackstone Publishing"
        },
        "length_minutes": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Audiobook runtime in whole minutes when known.",
          "example": 560
        },
        "published_at": {
          "type": [
            "string",
            "null"
          ],
          "description": "Publication date string from the catalog (ISO or storefront format).",
          "example": "2005-06-21"
        },
        "categories": {
          "type": [
            "string",
            "null"
          ],
          "description": "Genre / subject tags as a single delimited string when known.",
          "example": "Science Fiction & Fantasy; Science Fiction"
        },
        "language": {
          "type": [
            "string",
            "null"
          ],
          "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
          "example": "en"
        },
        "is_abridged": {
          "type": [
            "boolean",
            "null"
          ],
          "description": "`Some(true)` abridged / `Some(false)` unabridged when the storefront said so.",
          "example": false
        },
        "rating_overall": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Audible community overall rating (0–5).",
          "example": 4.5
        },
        "rating_performance": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Narration / performance sub-score when the storefront splits ratings.",
          "example": 4.7
        },
        "rating_story": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Story / content sub-score when the storefront splits ratings.",
          "example": 4.3
        },
        "rating_count": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Count of star ratings (Audible `num_ratings`).",
          "example": 12840
        },
        "review_count": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int64",
          "description": "Count of written reviews (Audible `num_reviews`).",
          "example": 1932
        },
        "reviews": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "One Audible customer review for the title detail panel.",
            "required": [
              "body"
            ],
            "properties": {
              "id": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Stable identifier for this item.",
                "example": "R3K2M9P1Q7X4Z"
              },
              "title": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display title as shown on the storefront or library card.",
                "example": "Still the best"
              },
              "body": {
                "type": "string",
                "description": "Full review text from the storefront.",
                "example": "Guidall's narration carries the quiet of Winter; I listened twice."
              },
              "author_name": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display name of the review author on the storefront.",
                "example": "Dana"
              },
              "overall_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Overall star rating for this review when present.",
                "example": 5
              },
              "performance_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Narration sub-rating for this review when present.",
                "example": 5
              },
              "story_rating": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Story sub-rating for this review when present.",
                "example": 4
              },
              "submitted_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "When the review was submitted (storefront timestamp string).",
                "example": "2026-02-11"
              }
            }
          },
          "description": "Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty\nfor older clients; prefer the paginated reviews endpoint."
        }
      }
    },
    {
      "type": "null"
    }
  ]
}
```

Status `400`: The query is malformed or too large.

```text
title, asin, or isbn is required
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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
      "asin": "B002V0QK4C",
      "title": "The Left Hand of Darkness",
      "subtitle": "A Hainish Novel",
      "authors": "Ursula K. Le Guin",
      "narrators": "George Guidall",
      "series": "Hainish Cycle",
      "series_index": "4",
      "isbn": "9780441478125",
      "cover_url": "https://images.example.com/covers/9780441478125.jpg",
      "description": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender.",
      "publisher": "Blackstone Publishing",
      "length_minutes": 560,
      "published_at": "2005-06-21",
      "categories": "Science Fiction & Fantasy; Science Fiction",
      "language": "en",
      "is_abridged": false,
      "rating_overall": 4.5,
      "rating_performance": 4.7,
      "rating_story": 4.3,
      "rating_count": 12840,
      "review_count": 1932,
      "reviews": [
        {
          "body": "Guidall's narration carries the quiet of Winter; I listened twice."
        }
      ]
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "Title-metadata results aligned with the request; failed items are `null`.",
  "required": [
    "results"
  ],
  "properties": {
    "results": {
      "type": "array",
      "items": {
        "oneOf": [
          {
            "type": "object",
            "description": "Bibliographic fields suitable for an Audible-style detail panel.",
            "properties": {
              "asin": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
                "example": "B002V0QK4C"
              },
              "title": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Display title as shown on the storefront or library card.",
                "example": "The Left Hand of Darkness"
              },
              "subtitle": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Optional subtitle when the catalog distinguishes it from the title.",
                "example": "A Hainish Novel"
              },
              "authors": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Comma-separated author names when the storefront provides them.",
                "example": "Ursula K. Le Guin"
              },
              "narrators": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Comma-separated narrator names when known.",
                "example": "George Guidall"
              },
              "series": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Series name when the title belongs to a named series.",
                "example": "Hainish Cycle"
              },
              "series_index": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Position within the series (e.g. `1`, `1.5`) when known.",
                "example": "4"
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
              "description": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publisher or storefront synopsis; may be truncated for embeddings.",
                "example": "An envoy to the planet Gethen must win the trust of a people who have no fixed gender."
              },
              "publisher": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publisher imprint as reported by the catalog.",
                "example": "Blackstone Publishing"
              },
              "length_minutes": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Audiobook runtime in whole minutes when known.",
                "example": 560
              },
              "published_at": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Publication date string from the catalog (ISO or storefront format).",
                "example": "2005-06-21"
              },
              "categories": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "Genre / subject tags as a single delimited string when known.",
                "example": "Science Fiction & Fantasy; Science Fiction"
              },
              "language": {
                "type": [
                  "string",
                  "null"
                ],
                "description": "BCP-47 / storefront language code (`en`, `de`, …) when known.",
                "example": "en"
              },
              "is_abridged": {
                "type": [
                  "boolean",
                  "null"
                ],
                "description": "`Some(true)` abridged / `Some(false)` unabridged when the storefront said so.",
                "example": false
              },
              "rating_overall": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Audible community overall rating (0–5).",
                "example": 4.5
              },
              "rating_performance": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Narration / performance sub-score when the storefront splits ratings.",
                "example": 4.7
              },
              "rating_story": {
                "type": [
                  "number",
                  "null"
                ],
                "format": "double",
                "description": "Story / content sub-score when the storefront splits ratings.",
                "example": 4.3
              },
              "rating_count": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Count of star ratings (Audible `num_ratings`).",
                "example": 12840
              },
              "review_count": {
                "type": [
                  "integer",
                  "null"
                ],
                "format": "int64",
                "description": "Count of written reviews (Audible `num_reviews`).",
                "example": 1932
              },
              "reviews": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "One Audible customer review for the title detail panel.",
                  "required": [
                    "body"
                  ],
                  "properties": {
                    "id": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Stable identifier for this item.",
                      "example": "R3K2M9P1Q7X4Z"
                    },
                    "title": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Display title as shown on the storefront or library card.",
                      "example": "Still the best"
                    },
                    "body": {
                      "type": "string",
                      "description": "Full review text from the storefront.",
                      "example": "Guidall's narration carries the quiet of Winter; I listened twice."
                    },
                    "author_name": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "Display name of the review author on the storefront.",
                      "example": "Dana"
                    },
                    "overall_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Overall star rating for this review when present.",
                      "example": 5
                    },
                    "performance_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Narration sub-rating for this review when present.",
                      "example": 5
                    },
                    "story_rating": {
                      "type": [
                        "integer",
                        "null"
                      ],
                      "format": "int64",
                      "description": "Story sub-rating for this review when present.",
                      "example": 4
                    },
                    "submitted_at": {
                      "type": [
                        "string",
                        "null"
                      ],
                      "description": "When the review was submitted (storefront timestamp string).",
                      "example": "2026-02-11"
                    }
                  }
                },
                "description": "Deprecated: reviews load via `/api/discover/title-reviews`. Kept empty\nfor older clients; prefer the paginated reviews endpoint."
              }
            }
          },
          {
            "type": "null"
          }
        ]
      },
      "description": "One optional metadata payload per query; enrichment errors become `None`."
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
at most 24 title-meta queries per batch
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

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
  - `sort_by` *string*, **required** — Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).
  - `reviews` *array of [`TitleReview`](schemas.md#titlereview)*, **required** — Review rows on this page.
- `400` — The query is malformed or too large. Body: string (`text/plain`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/discover/title-reviews' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"asin": "B002V0QK4C"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/discover/title-reviews' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"asin": "B002V0QK4C"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/discover/title-reviews",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"asin": "B002V0QK4C"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/discover/title-reviews", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"asin": "B002V0QK4C"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/discover/title-reviews")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"asin": "B002V0QK4C"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: A page of reviews.

```json
{
  "asin": "B002V0QK4C",
  "page": 1,
  "page_size": 10,
  "has_more": true,
  "sort_by": "MostHelpful",
  "reviews": [
    {
      "id": "R3K2M9P1Q7X4Z",
      "title": "Still the best",
      "body": "Guidall's narration carries the quiet of Winter; I listened twice.",
      "author_name": "Dana",
      "overall_rating": 5,
      "performance_rating": 5,
      "story_rating": 4,
      "submitted_at": "2026-02-11"
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "One page of title reviews for infinite scroll.",
  "required": [
    "asin",
    "page",
    "page_size",
    "has_more",
    "sort_by",
    "reviews"
  ],
  "properties": {
    "asin": {
      "type": "string",
      "description": "Audible / Amazon ASIN when this edition is sold on Audible.",
      "example": "B002V0QK4C"
    },
    "page": {
      "type": "integer",
      "format": "int32",
      "description": "1-based page index for this reviews / list request.",
      "example": 1,
      "minimum": 0
    },
    "page_size": {
      "type": "integer",
      "format": "int32",
      "description": "Maximum number of items returned in this page.",
      "example": 10,
      "minimum": 0
    },
    "has_more": {
      "type": "boolean",
      "description": "When true, another page is available by incrementing [`Self::page`].",
      "example": true
    },
    "sort_by": {
      "type": "string",
      "description": "Remote sort key for reviews (`MostHelpful` or `MostRecent`, as applied).",
      "example": "MostHelpful"
    },
    "reviews": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One Audible customer review for the title detail panel.",
        "required": [
          "body"
        ],
        "properties": {
          "id": {
            "type": [
              "string",
              "null"
            ],
            "description": "Stable identifier for this item.",
            "example": "R3K2M9P1Q7X4Z"
          },
          "title": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display title as shown on the storefront or library card.",
            "example": "Still the best"
          },
          "body": {
            "type": "string",
            "description": "Full review text from the storefront.",
            "example": "Guidall's narration carries the quiet of Winter; I listened twice."
          },
          "author_name": {
            "type": [
              "string",
              "null"
            ],
            "description": "Display name of the review author on the storefront.",
            "example": "Dana"
          },
          "overall_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Overall star rating for this review when present.",
            "example": 5
          },
          "performance_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Narration sub-rating for this review when present.",
            "example": 5
          },
          "story_rating": {
            "type": [
              "integer",
              "null"
            ],
            "format": "int64",
            "description": "Story sub-rating for this review when present.",
            "example": 4
          },
          "submitted_at": {
            "type": [
              "string",
              "null"
            ],
            "description": "When the review was submitted (storefront timestamp string).",
            "example": "2026-02-11"
          }
        }
      },
      "description": "Review rows on this page."
    }
  }
}
```

Status `400`: The query is malformed or too large.

```text
asin is required
```

Schema:

A `text/plain` body: a string.

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

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "forbidden",
  "message": "You do not have access to this resource.",
  "status": 403
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
