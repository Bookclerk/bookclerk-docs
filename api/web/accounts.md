# Accounts

Store accounts a library user connects (the Accounts page).

Back to the [web API reference](README.md).

## `POST /api/portal/redeem`

Redeems a claim ticket, optionally setting a first password, and sets the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`RedeemBody`](schemas.md#redeembody)

- `ticket` *string*, **required** — Raw claim-ticket token to redeem into a portal session.
- `nonce` *string*, **required** — Browser-generated one-time nonce persisted across HTTP retries.
- `password` *string or null* — Required when the linked local user has no password (invite / reset).

### Responses

- `200` — Signed in; the response sets the portal session cookie. Body: [`PortalOk`](schemas.md#portalok) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
- `400` — The ticket or nonce is invalid, used or expired, the ticket's integration is not deployed here, a first local claim needs a password of at least 8 characters, or the library refused the claim (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `503` — The library's atomic redeem did not answer in time or its reply was lost; redeem again with the same ticket and nonce. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/redeem' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"ticket": "3f9a1c7e5b2d4a6f8e0c1b3d5f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a", "nonce": "a7c3e9f1b5d2486e0f3a9c7b1d5e2f8a4c6b0d9e3f7a1c5b8d2e6f0a4c8b1d3e"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/redeem' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"ticket": "3f9a1c7e5b2d4a6f8e0c1b3d5f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a", "nonce": "a7c3e9f1b5d2486e0f3a9c7b1d5e2f8a4c6b0d9e3f7a1c5b8d2e6f0a4c8b1d3e"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/redeem",
    json={"ticket": "3f9a1c7e5b2d4a6f8e0c1b3d5f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a", "nonce": "a7c3e9f1b5d2486e0f3a9c7b1d5e2f8a4c6b0d9e3f7a1c5b8d2e6f0a4c8b1d3e"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/redeem", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"ticket": "3f9a1c7e5b2d4a6f8e0c1b3d5f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a", "nonce": "a7c3e9f1b5d2486e0f3a9c7b1d5e2f8a4c6b0d9e3f7a1c5b8d2e6f0a4c8b1d3e"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/redeem")
    .json(&serde_json::json!({"ticket": "3f9a1c7e5b2d4a6f8e0c1b3d5f7a9c2e4b6d8f0a1c3e5b7d9f2a4c6e8b0d1f3a", "nonce": "a7c3e9f1b5d2486e0f3a9c7b1d5e2f8a4c6b0d9e3f7a1c5b8d2e6f0a4c8b1d3e"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a portal route that has nothing to report beyond success.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
    }
  }
}
```

Status `400`: The ticket or nonce is invalid, used or expired, the ticket's integration is not deployed here, a first local claim needs a password of at least 8 characters, or the library refused the claim (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "integration `audiobookshelf` is not deployed on this host"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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
Failed to deserialize the JSON body into the target type: missing field `ticket` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `503`: The library's atomic redeem did not answer in time or its reply was lost; redeem again with the same ticket and nonce.

```json
{
  "error": "deadline_exceeded: host RPC deadline elapsed"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

## `POST /api/portal/login/integration`

Authenticates against an enabled integration and mints a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`IntegrationLoginBody`](schemas.md#integrationloginbody)

- `provider` *string*, **required** — Integration plugin id (must be enabled in config).
- `username` *string*, **required** — Provider login name forwarded to `authenticate_user`.
- `password` *string*, **required** — Provider password; never logged.

### Responses

- `200` — Signed in through the integration; the response sets the portal session cookie. Body: [`PortalOk`](schemas.md#portalok) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
- `400` — The integration is not deployed here, does not take a username and password, or refused them, or the library database refused the step. As `text/plain`, when the body is not syntactically valid JSON. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/login/integration' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"provider": "audiobookshelf", "username": "dana", "password": "correct horse battery staple"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/login/integration' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"provider": "audiobookshelf", "username": "dana", "password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/login/integration",
    json={"provider": "audiobookshelf", "username": "dana", "password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/login/integration", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"provider": "audiobookshelf", "username": "dana", "password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/login/integration")
    .json(&serde_json::json!({"provider": "audiobookshelf", "username": "dana", "password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in through the integration; the response sets the portal session cookie.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a portal route that has nothing to report beyond success.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
    }
  }
}
```

Status `400`: The integration is not deployed here, does not take a username and password, or refused them, or the library database refused the step. As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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
Failed to deserialize the JSON body into the target type: missing field `provider` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/logout`

Revokes the portal session row and clears the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — Signed out; the cookie is cleared. Body: [`PortalOk`](schemas.md#portalok) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/logout' \
    -X POST
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/logout' `
    -Method Post
```

```python
import requests

response = requests.post("http://127.0.0.1:8787/api/portal/logout")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/logout", {
  method: "POST",
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/logout")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed out; the cookie is cleared.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a portal route that has nothing to report beyond success.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/portal/me`

Returns the signed-in portal identity, or 401 when the cookie is missing or expired.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The signed-in identity. Body: [`MeResponse`](schemas.md#meresponse) (`application/json`)
  - `provider` *string*, **required** — Identity provider id (`local` or an integration plugin id).
  - `external_user_id` *string*, **required** — Provider-scoped user id stored on the portal identity row.
  - `label` *string or null* — Optional display label from the identity row.
- `400` — The library database refused the lookup. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/me' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/me' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/portal/me",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/me", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/portal/me")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The signed-in identity.

```json
{
  "provider": "local",
  "external_user_id": "user:7",
  "label": "Dana Reader"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /me`.",
  "required": [
    "provider",
    "external_user_id"
  ],
  "properties": {
    "provider": {
      "type": "string",
      "description": "Identity provider id (`local` or an integration plugin id).",
      "example": "local"
    },
    "external_user_id": {
      "type": "string",
      "description": "Provider-scoped user id stored on the portal identity row.",
      "example": "user:7"
    },
    "label": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional display label from the identity row.",
      "example": "Dana Reader"
    }
  }
}
```

Status `400`: The library database refused the lookup.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

## `GET /api/portal/sources`

Lists the deployed content sources (one per plugin, through its default
instance) with portal auth mode, config options, and brand colors.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — Deployed content sources. Body: [`SourcesResponse`](schemas.md#sourcesresponse) (`application/json`)
  - `sources` *array of [`SourceInfo`](schemas.md#sourceinfo)*, **required** — Enabled storefronts the SPA may offer for connect / login.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/sources'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/portal/sources")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/sources");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/portal/sources").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Deployed content sources.

```json
{
  "sources": [
    {
      "id": "libro",
      "name": "Libro.fm",
      "auth": "password",
      "config_options": [
        {
          "key": "container",
          "label": "Container",
          "values": []
        }
      ],
      "brand": {
        "bg": "#1F4E3D",
        "fg": "#F4F1EA",
        "accent": "#2F6B53",
        "logo": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /sources`.",
  "required": [
    "sources"
  ],
  "properties": {
    "sources": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One enabled storefront as shown on the portal Accounts page.",
        "required": [
          "id",
          "name",
          "auth",
          "brand"
        ],
        "properties": {
          "id": {
            "type": "string",
            "description": "Source plugin id (`audible`, `libro`, …).",
            "example": "libro"
          },
          "name": {
            "type": "string",
            "description": "Operator-facing storefront name.",
            "example": "Libro.fm"
          },
          "auth": {
            "type": "string",
            "description": "Portal auth mode (`password` or `oauth`).",
            "example": "password"
          },
          "config_options": {
            "type": "array",
            "items": {
              "type": "object",
              "description": "One storefront config option (key, label, allowed values).",
              "required": [
                "key",
                "label",
                "values"
              ],
              "properties": {
                "key": {
                  "type": "string",
                  "description": "Config table key (for example `container`).",
                  "example": "container"
                },
                "label": {
                  "type": "string",
                  "description": "Operator-facing option name.",
                  "example": "Container"
                },
                "values": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "description": "One allowed value for a storefront config option.",
                    "required": [
                      "id",
                      "label"
                    ],
                    "properties": {
                      "id": {
                        "type": "string",
                        "description": "Stored option value (for example `m4b`).",
                        "example": "m4b"
                      },
                      "label": {
                        "type": "string",
                        "description": "Operator-facing value label.",
                        "example": "M4B"
                      }
                    }
                  },
                  "description": "Allowed values the SPA may offer for this option."
                }
              }
            },
            "description": "Store-specific knobs the SPA can present (omitted when empty)."
          },
          "brand": {
            "type": "object",
            "description": "Storefront colors and logo href for the Accounts UI.",
            "required": [
              "bg",
              "fg",
              "accent",
              "logo"
            ],
            "properties": {
              "bg": {
                "type": "string",
                "description": "Background color (`#RRGGBB`).",
                "example": "#1F4E3D"
              },
              "fg": {
                "type": "string",
                "description": "Foreground / text color (`#RRGGBB`).",
                "example": "#F4F1EA"
              },
              "accent": {
                "type": "string",
                "description": "Accent color (`#RRGGBB`).",
                "example": "#2F6B53"
              },
              "logo": {
                "type": "string",
                "description": "Same-origin logo image (a 128 px copy of the brand's icon), or empty\nwhen there is none.",
                "example": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
              }
            }
          }
        }
      },
      "description": "Enabled storefronts the SPA may offer for connect / login."
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

## `POST /api/portal/sources/{id}/login`

Connects a password storefront, upserts the account, and links it to the signed-in identity.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `id` (path) *string*, **required** — Source plugin id.

### Request body

`application/json`: [`PasswordLoginBody`](schemas.md#passwordloginbody)

- `email` *string*, **required** — Store account email / username.
- `password` *string*, **required** — Store account password; never logged.

### Responses

- `200` — The account was connected. Body: [`SourceConnected`](schemas.md#sourceconnected) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
  - `account_id` *string*, **required** — The store's id for the account.
  - `source` *string*, **required** — The source's id, as listed by `GET /api/portal/sources`.
  - `plugin_key` *string*, **required** — The plugin that owns the account.
- `400` — No deployed source has that id, the source signs in through OAuth, or the store or the library database refused the step (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<id>/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "reader@example.com", "password": "correct horse battery staple"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/sources/<id>/login' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"email": "reader@example.com", "password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/sources/<id>/login",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"email": "reader@example.com", "password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/sources/<id>/login", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"email": "reader@example.com", "password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/sources/<id>/login")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"email": "reader@example.com", "password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The account was connected.

```json
{
  "ok": true,
  "account_id": "reader@example.com",
  "source": "libro",
  "plugin_key": "bookclerk:source-libro"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a connected store account.",
  "required": [
    "ok",
    "account_id",
    "source",
    "plugin_key"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
    },
    "account_id": {
      "type": "string",
      "description": "The store's id for the account.",
      "example": "reader@example.com"
    },
    "source": {
      "type": "string",
      "description": "The source's id, as listed by `GET /api/portal/sources`.",
      "example": "libro"
    },
    "plugin_key": {
      "type": "string",
      "description": "The plugin that owns the account.",
      "example": "bookclerk:source-libro"
    }
  }
}
```

Status `400`: No deployed source has that id, the source signs in through OAuth, or the store or the library database refused the step (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `email` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/sources/{id}/oauth/start`

Starts interactive OAuth for an OAuth storefront and returns the browser login URL.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `id` (path) *string*, **required** — Source plugin id.

### Responses

- `200` — The browser login URL. Body: [`OAuthStartUrl`](schemas.md#oauthstarturl) (`application/json`)
  - `url` *string*, **required** — The store's login page, bound to this sign-in.
- `400` — No deployed source has that id, the source signs in with a password, the store produced no login URL in time, the login stopped before it produced one, or the library database refused the step. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The browser login URL.

```json
{
  "url": "https://www.amazon.com/ap/signin?openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&…"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of an OAuth start: the browser login URL to open.",
  "required": [
    "url"
  ],
  "properties": {
    "url": {
      "type": "string",
      "description": "The store's login page, bound to this sign-in.",
      "example": "https://www.amazon.com/ap/signin?openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&…"
    }
  }
}
```

Status `400`: No deployed source has that id, the source signs in with a password, the store produced no login URL in time, the login stopped before it produced one, or the library database refused the step.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/libro/login`

**Deprecated.**

Legacy `POST /libro/login` alias that forwards to the Libro password-login handler.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasswordLoginBody`](schemas.md#passwordloginbody)

- `email` *string*, **required** — Store account email / username.
- `password` *string*, **required** — Store account password; never logged.

### Responses

- `200` — The account was connected. Body: [`SourceConnected`](schemas.md#sourceconnected) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
  - `account_id` *string*, **required** — The store's id for the account.
  - `source` *string*, **required** — The source's id, as listed by `GET /api/portal/sources`.
  - `plugin_key` *string*, **required** — The plugin that owns the account.
- `400` — The Libro.fm source is not deployed here, or the store or the library database refused the step (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`); string (`text/plain`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `415` — The request did not declare a JSON body. Body: string (`text/plain`)
- `422` — The JSON does not fit the body: a missing field, a wrong type or an unknown shape. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/libro/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "reader@example.com", "password": "correct horse battery staple"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/libro/login' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"email": "reader@example.com", "password": "correct horse battery staple"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/libro/login",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"email": "reader@example.com", "password": "correct horse battery staple"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/libro/login", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"email": "reader@example.com", "password": "correct horse battery staple"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/libro/login")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"email": "reader@example.com", "password": "correct horse battery staple"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The account was connected.

```json
{
  "ok": true,
  "account_id": "reader@example.com",
  "source": "libro",
  "plugin_key": "bookclerk:source-libro"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a connected store account.",
  "required": [
    "ok",
    "account_id",
    "source",
    "plugin_key"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
    },
    "account_id": {
      "type": "string",
      "description": "The store's id for the account.",
      "example": "reader@example.com"
    },
    "source": {
      "type": "string",
      "description": "The source's id, as listed by `GET /api/portal/sources`.",
      "example": "libro"
    },
    "plugin_key": {
      "type": "string",
      "description": "The plugin that owns the account.",
      "example": "bookclerk:source-libro"
    }
  }
}
```

Status `400`: The Libro.fm source is not deployed here, or the store or the library database refused the step (the body carries its message). As `text/plain`, when the body is not syntactically valid JSON.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

Status `415`: The request did not declare a JSON body.

```text
Expected request with `Content-Type: application/json`
```

Schema:

A `text/plain` body: a string.

Status `422`: The JSON does not fit the body: a missing field, a wrong type or an unknown shape.

```text
Failed to deserialize the JSON body into the target type: missing field `email` at line 1 column 2
```

Schema:

A `text/plain` body: a string.

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/audible/start`

**Deprecated.**

Legacy `POST /audible/start` alias that forwards to Audible OAuth start.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The browser login URL. Body: [`OAuthStartUrl`](schemas.md#oauthstarturl) (`application/json`)
  - `url` *string*, **required** — The store's login page, bound to this sign-in.
- `400` — The Audible source is not deployed here, the store produced no login URL in time, the login stopped before it produced one, or the library database refused the step. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/audible/start' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/audible/start' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/audible/start",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/audible/start", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/audible/start")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The browser login URL.

```json
{
  "url": "https://www.amazon.com/ap/signin?openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&…"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of an OAuth start: the browser login URL to open.",
  "required": [
    "url"
  ],
  "properties": {
    "url": {
      "type": "string",
      "description": "The store's login page, bound to this sign-in.",
      "example": "https://www.amazon.com/ap/signin?openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&…"
    }
  }
}
```

Status `400`: The Audible source is not deployed here, the store produced no login URL in time, the login stopped before it produced one, or the library database refused the step.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.

## `GET /api/portal/connections`

Lists store accounts linked to the signed-in identity, including disabled sources.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — Store accounts linked to the caller. Body: [`ConnectionsResponse`](schemas.md#connectionsresponse) (`application/json`)
  - `connections` *array of [`ConnectionInfo`](schemas.md#connectioninfo)*, **required** — Linked store accounts the SPA may show or revoke.
- `400` — The library database refused the lookup. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/connections' \
    -b 'bookclerk_portal_session=<portalSession>'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/connections' `
    -WebSession $session
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/portal/connections",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/connections", {
  headers: { Cookie: "bookclerk_portal_session=<portalSession>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/portal/connections")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Store accounts linked to the caller.

```json
{
  "connections": [
    {
      "account_id": "reader@example.com",
      "source": "libro",
      "plugin_key": "platform:bookclerk/bookclerk-plugin-source-libro#libro",
      "label": "Household",
      "connection_status": "active",
      "source_enabled": true,
      "brand": {
        "bg": "#1F4E3D",
        "fg": "#F4F1EA",
        "accent": "#2F6B53",
        "logo": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
      }
    }
  ]
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body for `GET /connections`.",
  "required": [
    "connections"
  ],
  "properties": {
    "connections": {
      "type": "array",
      "items": {
        "type": "object",
        "description": "One linked store account on the portal Accounts page.",
        "required": [
          "account_id",
          "source",
          "plugin_key",
          "connection_status",
          "source_enabled"
        ],
        "properties": {
          "account_id": {
            "type": "string",
            "description": "Store account id (unique within its source).",
            "example": "reader@example.com"
          },
          "source": {
            "type": "string",
            "description": "Source plugin id that owns this account.",
            "example": "libro"
          },
          "plugin_key": {
            "type": "string",
            "description": "Provenance-qualified PluginKey that owns the account.",
            "example": "platform:bookclerk/bookclerk-plugin-source-libro#libro"
          },
          "label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing account label when the store provided one.",
            "example": "Household"
          },
          "connection_status": {
            "type": "string",
            "description": "Connection state (`active`, revoked, …); defaults to `active` if the account row is missing.",
            "example": "active"
          },
          "source_enabled": {
            "type": "boolean",
            "description": "Whether `[sources.<id>] enabled` is currently true (new connects blocked when false).",
            "example": true
          },
          "brand": {
            "oneOf": [
              {
                "type": "object",
                "description": "Storefront or integration brand when a plugin or fallback palette is available.",
                "required": [
                  "bg",
                  "fg",
                  "accent",
                  "logo"
                ],
                "properties": {
                  "bg": {
                    "type": "string",
                    "description": "Background color (`#RRGGBB`).",
                    "example": "#1F4E3D"
                  },
                  "fg": {
                    "type": "string",
                    "description": "Foreground / text color (`#RRGGBB`).",
                    "example": "#F4F1EA"
                  },
                  "accent": {
                    "type": "string",
                    "description": "Accent color (`#RRGGBB`).",
                    "example": "#2F6B53"
                  },
                  "logo": {
                    "type": "string",
                    "description": "Same-origin logo image (a 128 px copy of the brand's icon), or empty\nwhen there is none.",
                    "example": "/api/images/aHR0cHM6Ly93d3cuZ29vZ2xlLmNvbS9zMi9mYXZpY29ucz9kb21haW49bGlicm8uZm0mc3o9MTI4.N14rqh66Bok9-eCqJKBNoQ"
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
      "description": "Linked store accounts the SPA may show or revoke."
    }
  }
}
```

Status `400`: The library database refused the lookup.

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

## `POST /api/portal/sources/{source}/connections/{account_id}/revoke`

Unlinks a store account from this identity and deletes secrets when no other identity remains.

The account is addressed by `(source, account_id)`: the same store id may
exist under two plugins. `source` is the alias or plugin key of the link.

When the last link goes, the plugin first signs the account out at the
store (for example, Audible deregisters its device), best effort.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

- `source` (path) *string*, **required** — Source plugin id.
- `account_id` (path) *string*, **required** — Store account id.

### Responses

- `200` — The account was unlinked. Body: [`PortalOk`](schemas.md#portalok) (`application/json`)
  - `ok` *boolean*, **required** — Always `true`.
- `400` — The account is not linked to this identity, or the library database refused the unlink or the credential revoke (the body carries its message). Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in, or the portal session expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `503` — The store sign-out task that follows the last unlink stopped. The link is already gone (a retry answers 400 `account not linked to this identity`) and the account's stored credentials may remain; the daemon log names the task. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The account was unlinked.

```json
{
  "ok": true
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a portal route that has nothing to report beyond success.",
  "required": [
    "ok"
  ],
  "properties": {
    "ok": {
      "type": "boolean",
      "description": "Always `true`.",
      "example": true
    }
  }
}
```

Status `400`: The account is not linked to this identity, or the library database refused the unlink or the credential revoke (the body carries its message).

```json
{
  "error": "ORM / database plugin error: Execution Error: error returned from database: database is locked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
    }
  }
}
```

Status `401`: The caller is not signed in, or the portal session expired.

```json
{
  "error": "not signed in"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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

Status `503`: The store sign-out task that follows the last unlink stopped. The link is already gone (a retry answers 400 `account not linked to this identity`) and the account's stored credentials may remain; the daemon log names the task.

```json
{
  "error": "revoke cleanup failed: task 12 panicked"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of every portal error response.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Operator-facing error text (no structured code).",
      "example": "not signed in"
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
