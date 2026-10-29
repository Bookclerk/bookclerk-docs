# Spec

This document.

Back to the [web API reference](README.md).

## `GET /api/openapi.json`

Serves the OpenAPI document.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The OpenAPI 3.1 document for this daemon's version. Body: [`OpenApiDocument`](schemas.md#openapidocument) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/openapi.json'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/openapi.json'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/openapi.json")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/openapi.json");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/openapi.json").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The OpenAPI 3.1 document for this daemon's version.

```json
{
  "components": {
    "schemas": {}
  },
  "info": {
    "title": "Bookclerk daemon API",
    "version": "0.1.0"
  },
  "openapi": "3.1.0",
  "paths": {
    "/health": {
      "get": {
        "responses": {
          "200": {
            "description": "The daemon is up."
          }
        }
      }
    }
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "An OpenAPI document as this daemon serves it: the `openapi` version string, `info` about the daemon, every route under `paths` and the schemas under `components`.",
  "examples": [
    {
      "components": {
        "schemas": {}
      },
      "info": {
        "title": "Bookclerk daemon API",
        "version": "0.1.0"
      },
      "openapi": "3.1.0",
      "paths": {
        "/health": {
          "get": {
            "responses": {
              "200": {
                "description": "The daemon is up."
              }
            }
          }
        }
      }
    }
  ]
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```

## `GET /api/openapi-3.0.json`

Serves the OpenAPI document restated as OpenAPI 3.0.3.

The same API as `GET /api/openapi.json`, restated in OpenAPI 3.0.3 form for tools that only read 3.0, such as Cloudflare API Shield's schema validation: nullable types use `nullable: true`, and a nullable reference is a copy of the schema it names.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The OpenAPI 3.0.3 document for this daemon's version. Body: [`OpenApiDocument`](schemas.md#openapidocument) (`application/json`)
- `500` — The document could not be restated; this is a bug. Body: [`OpenApiV30Unavailable`](schemas.md#openapiv30unavailable) (`application/json`)
  - `error` *string*, **required** — Always `openapi_v30_unavailable`.
  - `message` *string*, **required** — What to do: the daemon log names the form that could not be restated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/openapi-3.0.json'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/openapi-3.0.json'
```

```python
import requests

response = requests.get("http://127.0.0.1:8787/api/openapi-3.0.json")
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/openapi-3.0.json");
console.log(response.status, await response.text());
```

```rust
let response = reqwest::get("http://127.0.0.1:8787/api/openapi-3.0.json").await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The OpenAPI 3.0.3 document for this daemon's version.

```json
{
  "components": {
    "schemas": {}
  },
  "info": {
    "title": "Bookclerk daemon API",
    "version": "0.1.0"
  },
  "openapi": "3.1.0",
  "paths": {
    "/health": {
      "get": {
        "responses": {
          "200": {
            "description": "The daemon is up."
          }
        }
      }
    }
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "An OpenAPI document as this daemon serves it: the `openapi` version string, `info` about the daemon, every route under `paths` and the schemas under `components`.",
  "examples": [
    {
      "components": {
        "schemas": {}
      },
      "info": {
        "title": "Bookclerk daemon API",
        "version": "0.1.0"
      },
      "openapi": "3.1.0",
      "paths": {
        "/health": {
          "get": {
            "responses": {
              "200": {
                "description": "The daemon is up."
              }
            }
          }
        }
      }
    }
  ]
}
```

Status `500`: The document could not be restated; this is a bug.

```json
{
  "error": "openapi_v30_unavailable",
  "message": "The OpenAPI 3.0 document could not be built; see the daemon log."
}
```

Schema:

```json
{
  "type": "object",
  "description": "Body of the `500` from `GET /api/openapi-3.0.json` when the 3.1\ndocument could not be restated as 3.0.3.",
  "required": [
    "error",
    "message"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `openapi_v30_unavailable`.",
      "example": "openapi_v30_unavailable"
    },
    "message": {
      "type": "string",
      "description": "What to do: the daemon log names the form that could not be restated.",
      "example": "The OpenAPI 3.0 document could not be built; see the daemon log."
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
string
```

Schema:

```json
{
  "type": "string"
}
```
