# Spec

This document.

Back to the [web API reference](README.md).

## `GET /api/openapi.json`

Serves the OpenAPI document.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The OpenAPI 3.1 document for this daemon's version. Body: any JSON (`application/json`)
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

## `GET /api/openapi-3.0.json`

Serves the OpenAPI document restated as OpenAPI 3.0.3.

The same API as `GET /api/openapi.json`, restated in OpenAPI 3.0.3 form for tools that only read 3.0, such as Cloudflare API Shield's schema validation: nullable types use `nullable: true`, and a nullable reference is a copy of the schema it names.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

- `200` — The OpenAPI 3.0.3 document for this daemon's version. Body: any JSON (`application/json`)
- `500` — The document could not be restated; this is a bug. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
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

Status `200`: The OpenAPI 3.0.3 document for this daemon's version. The body is JSON the schemas do not describe yet.

Status `500`: The document could not be restated; this is a bug.

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
