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

- `200` — Signed in; the response sets the portal session cookie. Body: any JSON (`application/json`)
- `400` — The ticket is invalid, used or expired. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/redeem' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"ticket": "string", "nonce": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/redeem' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"ticket": "string", "nonce": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/redeem",
    json={"ticket": "string", "nonce": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/redeem", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"ticket": "string", "nonce": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/redeem")
    .json(&serde_json::json!({"ticket": "string", "nonce": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie. The body is JSON the schemas do not describe yet.

Status `400`: The ticket is invalid, used or expired.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
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

- `200` — Signed in through the integration. Body: any JSON (`application/json`)
- `400` — The request is invalid, or the integration refused the login. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/login/integration' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"provider": "string", "username": "string", "password": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/login/integration' `
    -Method Post `
    -ContentType 'application/json' `
    -Body '{"provider": "string", "username": "string", "password": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/login/integration",
    json={"provider": "string", "username": "string", "password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/login/integration", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({"provider": "string", "username": "string", "password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/login/integration")
    .json(&serde_json::json!({"provider": "string", "username": "string", "password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Signed in through the integration. The body is JSON the schemas do not describe yet.

Status `400`: The request is invalid, or the integration refused the login.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/logout`

Revokes the portal session row and clears the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — Signed out (`{"ok": true}`); the cookie is cleared. Body: any JSON (`application/json`)
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

Status `200`: Signed out (`{"ok": true}`); the cookie is cleared. The body is JSON the schemas do not describe yet.

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

Status `504`: The request ran past the API's time budget.

```text
string
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
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
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
  "provider": "string",
  "external_user_id": "string",
  "label": "string"
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
      "description": "Identity provider id (`local` or an integration plugin id)."
    },
    "external_user_id": {
      "type": "string",
      "description": "Provider-scoped user id stored on the portal identity row."
    },
    "label": {
      "type": [
        "string",
        "null"
      ],
      "description": "Optional display label from the identity row."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
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
      "id": "string",
      "name": "string",
      "auth": "string",
      "config_options": [
        {
          "key": "string",
          "label": "string",
          "values": []
        }
      ],
      "brand": {
        "bg": "string",
        "fg": "string",
        "accent": "string",
        "logo": "string"
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
            "description": "Source plugin id (`audible`, `libro`, …)."
          },
          "name": {
            "type": "string",
            "description": "Operator-facing storefront name."
          },
          "auth": {
            "type": "string",
            "description": "Portal auth mode (`password` or `oauth`)."
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
                  "description": "Config table key (for example `container`)."
                },
                "label": {
                  "type": "string",
                  "description": "Operator-facing option name."
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
                        "description": "Stored option value (for example `m4b`)."
                      },
                      "label": {
                        "type": "string",
                        "description": "Operator-facing value label."
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
                "description": "Background color (`#RRGGBB`)."
              },
              "fg": {
                "type": "string",
                "description": "Foreground / text color (`#RRGGBB`)."
              },
              "accent": {
                "type": "string",
                "description": "Accent color (`#RRGGBB`)."
              },
              "logo": {
                "type": "string",
                "description": "Logo image href (often a favicon URL)."
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
string
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

- `200` — The account was connected. Body: any JSON (`application/json`)
- `400` — The login failed. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<id>/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "string", "password": "string"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/sources/<id>/login' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"email": "string", "password": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/sources/<id>/login",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"email": "string", "password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/sources/<id>/login", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"email": "string", "password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/sources/<id>/login")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"email": "string", "password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The account was connected. The body is JSON the schemas do not describe yet.

Status `400`: The login failed.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
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
string
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

- `200` — The browser login URL. Body: any JSON (`application/json`)
- `400` — The source does not use OAuth. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
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

Status `200`: The browser login URL. The body is JSON the schemas do not describe yet.

Status `400`: The source does not use OAuth.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
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
string
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

- `200` — The account was connected. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/libro/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "string", "password": "string"}'
```

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.Cookies.Add((New-Object System.Net.Cookie('bookclerk_portal_session', '<portalSession>', '/', '127.0.0.1')))
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/portal/libro/login' `
    -Method Post `
    -WebSession $session `
    -Headers @{ Origin = 'http://127.0.0.1:8787' } `
    -ContentType 'application/json' `
    -Body '{"email": "string", "password": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/portal/libro/login",
    headers={"Cookie": "bookclerk_portal_session=<portalSession>", "Origin": "http://127.0.0.1:8787"},
    json={"email": "string", "password": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/portal/libro/login", {
  method: "POST",
  headers: { Cookie: "bookclerk_portal_session=<portalSession>", Origin: "http://127.0.0.1:8787", "Content-Type": "application/json" },
  body: JSON.stringify({"email": "string", "password": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/portal/libro/login")
    .header("Cookie", "bookclerk_portal_session=<portalSession>")
    .header("Origin", "http://127.0.0.1:8787")
    .json(&serde_json::json!({"email": "string", "password": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The account was connected. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
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
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/portal/audible/start`

**Deprecated.**

Legacy `POST /audible/start` alias that forwards to Audible OAuth start.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

- `200` — The browser login URL. Body: any JSON (`application/json`)
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
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

Status `200`: The browser login URL. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
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
string
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
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
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
      "account_id": "string",
      "source": "string",
      "plugin_key": "string",
      "label": "string",
      "connection_status": "string",
      "source_enabled": true,
      "brand": {
        "bg": "string",
        "fg": "string",
        "accent": "string",
        "logo": "string"
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
            "description": "Store account id (unique within its source)."
          },
          "source": {
            "type": "string",
            "description": "Source plugin id that owns this account."
          },
          "plugin_key": {
            "type": "string",
            "description": "Provenance-qualified PluginKey that owns the account."
          },
          "label": {
            "type": [
              "string",
              "null"
            ],
            "description": "Operator-facing account label when the store provided one."
          },
          "connection_status": {
            "type": "string",
            "description": "Connection state (`active`, revoked, …); defaults to `active` if the account row is missing."
          },
          "source_enabled": {
            "type": "boolean",
            "description": "Whether `[sources.<id>] enabled` is currently true (new connects blocked when false)."
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
                    "description": "Background color (`#RRGGBB`)."
                  },
                  "fg": {
                    "type": "string",
                    "description": "Foreground / text color (`#RRGGBB`)."
                  },
                  "accent": {
                    "type": "string",
                    "description": "Accent color (`#RRGGBB`)."
                  },
                  "logo": {
                    "type": "string",
                    "description": "Logo image href (often a favicon URL)."
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

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
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

- `200` — The account was unlinked. Body: any JSON (`application/json`)
- `400` — The account is not linked to this identity. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `401` — The caller is not signed in. Body: [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`)
  - `error` *string*, **required** — Operator-facing error text (no structured code).
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
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

Status `200`: The account was unlinked. The body is JSON the schemas do not describe yet.

Status `400`: The account is not linked to this identity.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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
      "description": "Operator-facing error text (no structured code)."
    }
  }
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
string
```

Schema:

A `text/plain` body: a string.
