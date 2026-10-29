# Plugins

Plugin packages, instances, consent and deployment.

Back to the [web API reference](README.md).

## `GET /api/config/plugin-instances`

`GET /api/config/plugin-instances`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Every plugin instance with its deployments. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugin-instances",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugin-instances")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Every plugin instance with its deployments. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route.

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

Status `500`: The configuration could not be read or written.

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

## `POST /api/config/plugin-instances`

`POST /api/config/plugin-instances`: a second (non-default) instance of
a plugin, with an empty document so schema defaults apply. With
`node_id`, the instance is local to that node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CreatePluginInstance`](schemas.md#createplugininstance)

- `plugin_key` *string*, **required** — Canonical plugin key; the plugin must have a pinned schema.
- `display_name` *string* — Operator-facing name.
- `node_id` *string or null* — Make the instance local to this node (H-2): it deploys only there and is never placed by the scheduler.

### Responses

- `200` — The new instance. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The instance id is taken. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"plugin_key": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"plugin_key": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugin-instances",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"plugin_key": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"plugin_key": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugin-instances")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"plugin_key": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The new instance. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such plugin.

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

Status `409`: The instance id is taken.

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

Status `500`: The configuration could not be read or written.

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

## `DELETE /api/config/plugin-instances/{id}`

`DELETE /api/config/plugin-instances/{id}`: removes an instance that has
no deployment left.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance was removed. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The instance is a default or still deployed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugin-instances/<id>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance was removed. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such instance.

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

Status `409`: The instance is a default or still deployed.

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

Status `500`: The configuration could not be read or written.

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

## `GET /api/config/plugin-instances/{id}/config`

`GET /api/config/plugin-instances/{id}/config`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance's settings document (secrets as presence flags). Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/config",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance's settings document (secrets as presence flags). The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route.

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

Status `404`: No such instance.

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

Status `500`: The configuration could not be read or written.

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

## `PUT /api/config/plugin-instances/{id}/config`

`PUT /api/config/plugin-instances/{id}/config`.

Operator principal only. The route is mounted on the operator router.
The document is validated against the plugin's pinned schema, so this
node does not need the guest installed.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Request body

`application/json`: [`PutPluginInstanceConfig`](schemas.md#putplugininstanceconfig)

- `expected_revision` *integer (int64)*, **required** — Revision the caller last observed.
- `settings` *object* — Scalar settings. Secret values are not accepted here.
- `secret_refs` *array of object* — Secret refs to keep. Omitted refs are dropped from the document; the ciphertext stays sealed until the secret is overwritten.
- `secrets` *map of string to string* — Secret values to seal, keyed by the declared `secret` setting. Each is sealed into `encrypted_secrets` and referenced from the document; the plaintext is never stored in the document or echoed back.

### Responses

- `200` — The document after the change. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"expected_revision": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/config' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"expected_revision": 0}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/config",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"expected_revision": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"expected_revision": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/config")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"expected_revision": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The document after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such instance.

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

Status `500`: The configuration could not be read or written.

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

## `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
Deployed switch. Creates the row when the node has none.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.
- `node_id` (path) *string*, **required** — Node id.

### Request body

`application/json`: [`PutPluginDeployment`](schemas.md#putplugindeployment)

- `desired` *[`DesiredState`](schemas.md#desiredstate)*, **required** — What the node should do: `present` or `absent` (the scheduler's `placed` / `unplaced` are accepted too).
- `expected_revision` *integer (int64) or null* — Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current".

### Responses

- `200` — The deployment after the change. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The instance is placed by the scheduler. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"desired": "present"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"desired": "present"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"desired": "present"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"desired": "present"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/deployments/<node_id>")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"desired": "present"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The deployment after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such instance.

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

Status `409`: The instance is placed by the scheduler.

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

Status `500`: The configuration could not be read or written.

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

## `PUT /api/config/plugin-instances/{id}/placement`

`PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
deployments to the scheduler with a replica count and constraints.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Request body

`application/json`: [`PutPluginPlacement`](schemas.md#putpluginplacement)

- `replicas` *integer or `all`*, **required** — A replica count (0 stops every replica) or `"all"`.
- `constraints` *array of string* — Label constraints (`name`, `name=value`, `!name`).
- `expected_revision` *integer (int64) or null* — Current revision to replace; omit (or 0) to create.

### Responses

- `200` — The scheduler now places the instance. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"replicas": 0}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"replicas": 0}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"replicas": 0},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"replicas": 0}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"replicas": 0}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The scheduler now places the instance. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such instance.

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

Status `500`: The configuration could not be read or written.

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

## `DELETE /api/config/plugin-instances/{id}/placement`

`DELETE /api/config/plugin-instances/{id}/placement`: return the
instance to hand-managed deployments; its deployments stay as they
are.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin instance id.

### Responses

- `200` — The instance is back to manual deployment. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such instance. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugin-instances/<id>/placement")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The instance is back to manual deployment. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such instance.

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

Status `500`: The configuration could not be read or written.

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

## `GET /api/config/plugins`

`GET /api/config/plugins`: every installed plugin.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — Every installed plugin. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugins",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugins")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: Every installed plugin. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route.

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

Status `500`: The configuration could not be read or written.

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

## `POST /api/config/plugins`

`POST /api/config/plugins`: install (and maybe activate) a version.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PostPluginInstall`](schemas.md#postplugininstall)

- `coordinate` *string or null* — Registry coordinate (`cargo:…@v`, `registry:…#name@v`).
- `package` *string or null* — Operator-placed package directory under `plugin-packages/`.
- `activate` *boolean* — Make the new version active even when another is.
- `approve` *boolean* — Record the requested consent as approved.
- `allow_unverified_publisher` *boolean* — Accept a publisher without independent authenticity.

### Responses

- `200` — The installed version. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — A local package is missing its `package.json` or `archive.tar.gz`. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `409` — The version is already installed. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `502` — The package catalog could not be reached or answered badly. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugins",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugins")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The installed version. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: A local package is missing its `package.json` or `archive.tar.gz`.

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

Status `409`: The version is already installed.

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

Status `500`: The configuration could not be read or written.

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

Status `502`: The package catalog could not be reached or answered badly.

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

## `GET /api/config/plugins/{key}`

`GET /api/config/plugins/{key}`: one installed plugin with its pinned
settings schema.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Responses

- `200` — The plugin with its pinned versions. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/config/plugins/<key>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/config/plugins/<key>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The plugin with its pinned versions. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route.

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

Status `404`: No such plugin.

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

Status `500`: The configuration could not be read or written.

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

## `DELETE /api/config/plugins/{key}`

`DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
cleared, placements dropped, every deployment `absent`).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Responses

- `200` — The plugin was uninstalled. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>' `
    -Method Delete `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.delete(
    "http://127.0.0.1:8787/api/config/plugins/<key>",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>", {
  method: "DELETE",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .delete("http://127.0.0.1:8787/api/config/plugins/<key>")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The plugin was uninstalled. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such plugin.

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

Status `500`: The configuration could not be read or written.

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

## `PUT /api/config/plugins/{key}/active`

`PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.

### Request body

`application/json`: [`PutActiveVersion`](schemas.md#putactiveversion)

- `version` *string*, **required** — Installed version to activate.

### Responses

- `200` — The active version after the change. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>/active' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"version": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>/active' `
    -Method Put `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"version": "string"}'
```

```python
import requests

response = requests.put(
    "http://127.0.0.1:8787/api/config/plugins/<key>/active",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"version": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>/active", {
  method: "PUT",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"version": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .put("http://127.0.0.1:8787/api/config/plugins/<key>/active")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"version": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The active version after the change. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such plugin.

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

Status `500`: The configuration could not be read or written.

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

## `POST /api/config/plugins/{key}/versions/{version}/approve`

`POST /api/config/plugins/{key}/versions/{version}/approve`: record the
version's requested consent as approved for every node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `key` (path) *string*, **required** — Plugin key.
- `version` (path) *string*, **required** — Semver version.

### Responses

- `200` — The version's grant was approved. Body: any JSON (`application/json`)
- `400` — The body is invalid. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `500` — The configuration could not be read or written. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/config/plugins/<key>/versions/<version>/approve")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The version's grant was approved. The body is JSON the schemas do not describe yet.

Status `400`: The body is invalid.

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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such plugin.

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

Status `500`: The configuration could not be read or written.

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

## `GET /api/plugins/{id}/consent`

Loads the consent dialog for one plugin, including coverage against the stored grant.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin id.
- `plugin_key` (query) *string* — Canonical PluginKey; when set, wins over the path alias.

### Responses

- `200` — What the plugin asks for and what is granted. Body: [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`)
  - `plugin_id` *string*, **required** — Plugin id whose grant is being reviewed.
  - `plugin_key` *string*, **required** — Canonical PluginKey for this install (empty when unknown).
  - `provenance` *string*, **required** — Host-evaluated provenance label.
  - `runtime` *string*, **required** — Guest runtime (`native` or `workerd`) — drives which controls are enforceable.
  - `request` *[`PluginGrantView`](schemas.md#plugingrantview)*, **required** — Manifest-derived grant the plugin is asking for.
  - `covered` *boolean*, **required** — True when an existing on-disk grant already satisfies the request.
  - `summary` *array of string*, **required** — Operator-facing bullet list of requested capabilities.
  - `existing` *[`PluginGrantView`](schemas.md#plugingrantview) or null* — Previously approved grant, when one is stored.
  - `brand` *[`PluginConsentBrand`](schemas.md#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
  - `limits` *[`PluginConsentLimits`](schemas.md#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The grant could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<id>/consent' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<id>/consent' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/plugins/<id>/consent",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<id>/consent", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/plugins/<id>/consent")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: What the plugin asks for and what is granted.

```json
{
  "plugin_id": "string",
  "plugin_key": "string",
  "provenance": "string",
  "runtime": "string",
  "request": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "covered": true,
  "summary": [
    "string"
  ],
  "existing": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "brand": {
    "name": "string",
    "bg": "string",
    "fg": "string",
    "accent": "string",
    "logo": "string"
  },
  "limits": {
    "cpu_ms": 0,
    "subrequests": 0,
    "max_cpu_ms": 0,
    "max_subrequests": 0,
    "disk_mib": 0,
    "max_disk_mib": 0,
    "memory_mib": 0,
    "max_memory_mib": 0,
    "cpu_cores": 0,
    "max_cpu_cores": 0,
    "jail_cpu_cores": 0,
    "extra_processes": 0,
    "max_extra_processes": 0,
    "known_bindings": [
      "string"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Consent dialog payload for one plugin.",
  "required": [
    "plugin_id",
    "plugin_key",
    "provenance",
    "runtime",
    "request",
    "covered",
    "summary",
    "limits"
  ],
  "properties": {
    "plugin_id": {
      "type": "string",
      "description": "Plugin id whose grant is being reviewed."
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical PluginKey for this install (empty when unknown)."
    },
    "provenance": {
      "type": "string",
      "description": "Host-evaluated provenance label."
    },
    "runtime": {
      "type": "string",
      "description": "Guest runtime (`native` or `workerd`) — drives which controls are enforceable."
    },
    "request": {
      "type": "object",
      "description": "Manifest-derived grant the plugin is asking for.",
      "required": [
        "pluginKey",
        "pluginId",
        "entrypoints",
        "producers",
        "consumers",
        "jobs",
        "networkMode",
        "domains",
        "manifestDomains",
        "operatorAddedDomains",
        "operatorDeniedDomains",
        "tcp",
        "operatorAddedTcp",
        "operatorDeniedTcp",
        "addressCidrs",
        "operatorAddedCidrs",
        "operatorDeniedCidrs",
        "allowUndeclaredPublicRedirects",
        "grantRevision",
        "authorityRevision",
        "bindings",
        "compatibilityFlags",
        "approvedAt"
      ],
      "properties": {
        "pluginKey": {
          "type": "string",
          "description": "Canonical provenance-qualified PluginKey."
        },
        "pluginId": {
          "type": "string",
          "description": "Plugin id this grant applies to."
        },
        "entrypoints": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
        },
        "producers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Event types the operator approved the plugin to publish."
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "object"
          },
          "description": "Approved event-consumer subscriptions."
        },
        "jobs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Approved job trigger types."
        },
        "networkMode": {
          "type": "string",
          "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
        },
        "domains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Hostnames the operator approved for egress."
        },
        "manifestDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Destinations requested by the current manifest."
        },
        "operatorAddedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added destinations beyond the manifest."
        },
        "operatorDeniedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied destinations still listed by the manifest."
        },
        "tcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Effective raw TCP grants."
        },
        "operatorAddedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-added TCP grants."
        },
        "operatorDeniedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-denied TCP grants still listed by the manifest."
        },
        "addressCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Effective address-space CIDRs."
        },
        "operatorAddedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added CIDRs."
        },
        "operatorDeniedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied CIDRs still listed by the manifest."
        },
        "allowUndeclaredPublicRedirects": {
          "type": "boolean",
          "description": "Undeclared public redirect permission."
        },
        "grantRevision": {
          "type": "string",
          "description": "SHA-256 of persisted operator consent (no host overlays)."
        },
        "authorityRevision": {
          "type": "string",
          "description": "SHA-256 of the canonical effective authority."
        },
        "bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Host bindings the operator approved."
        },
        "compatibilityFlags": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Extra compatibility flags stored on the grant."
        },
        "approvedAt": {
          "type": "string",
          "description": "RFC 3339 timestamp when the grant was last approved."
        },
        "cpuMs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd subrequest budget.",
          "minimum": 0
        },
        "diskMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "memoryMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional jail memory in mebibytes.",
          "minimum": 0
        },
        "cpuCores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Jail CPU in cores (2 d.p.); absent for workerd."
        },
        "extraProcesses": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Extra process/thread budget beyond launcher overhead; native only.",
          "minimum": 0
        }
      }
    },
    "covered": {
      "type": "boolean",
      "description": "True when an existing on-disk grant already satisfies the request."
    },
    "summary": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Operator-facing bullet list of requested capabilities."
    },
    "existing": {
      "oneOf": [
        {
          "type": "object",
          "description": "Previously approved grant, when one is stored.",
          "required": [
            "pluginKey",
            "pluginId",
            "entrypoints",
            "producers",
            "consumers",
            "jobs",
            "networkMode",
            "domains",
            "manifestDomains",
            "operatorAddedDomains",
            "operatorDeniedDomains",
            "tcp",
            "operatorAddedTcp",
            "operatorDeniedTcp",
            "addressCidrs",
            "operatorAddedCidrs",
            "operatorDeniedCidrs",
            "allowUndeclaredPublicRedirects",
            "grantRevision",
            "authorityRevision",
            "bindings",
            "compatibilityFlags",
            "approvedAt"
          ],
          "properties": {
            "pluginKey": {
              "type": "string",
              "description": "Canonical provenance-qualified PluginKey."
            },
            "pluginId": {
              "type": "string",
              "description": "Plugin id this grant applies to."
            },
            "entrypoints": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
            },
            "producers": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Event types the operator approved the plugin to publish."
            },
            "consumers": {
              "type": "array",
              "items": {
                "type": "object"
              },
              "description": "Approved event-consumer subscriptions."
            },
            "jobs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Approved job trigger types."
            },
            "networkMode": {
              "type": "string",
              "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
            },
            "domains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Hostnames the operator approved for egress."
            },
            "manifestDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Destinations requested by the current manifest."
            },
            "operatorAddedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added destinations beyond the manifest."
            },
            "operatorDeniedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied destinations still listed by the manifest."
            },
            "tcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Effective raw TCP grants."
            },
            "operatorAddedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-added TCP grants."
            },
            "operatorDeniedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-denied TCP grants still listed by the manifest."
            },
            "addressCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Effective address-space CIDRs."
            },
            "operatorAddedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added CIDRs."
            },
            "operatorDeniedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied CIDRs still listed by the manifest."
            },
            "allowUndeclaredPublicRedirects": {
              "type": "boolean",
              "description": "Undeclared public redirect permission."
            },
            "grantRevision": {
              "type": "string",
              "description": "SHA-256 of persisted operator consent (no host overlays)."
            },
            "authorityRevision": {
              "type": "string",
              "description": "SHA-256 of the canonical effective authority."
            },
            "bindings": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Host bindings the operator approved."
            },
            "compatibilityFlags": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Extra compatibility flags stored on the grant."
            },
            "approvedAt": {
              "type": "string",
              "description": "RFC 3339 timestamp when the grant was last approved."
            },
            "cpuMs": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd CPU budget in milliseconds.",
              "minimum": 0
            },
            "subrequests": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd subrequest budget.",
              "minimum": 0
            },
            "diskMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional plugin state disk budget in mebibytes.",
              "minimum": 0
            },
            "memoryMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional jail memory in mebibytes.",
              "minimum": 0
            },
            "cpuCores": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Jail CPU in cores (2 d.p.); absent for workerd."
            },
            "extraProcesses": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Extra process/thread budget beyond launcher overhead; native only.",
              "minimum": 0
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "brand": {
      "oneOf": [
        {
          "type": "object",
          "description": "Optional portal or manifest branding for the dialog.",
          "required": [
            "name"
          ],
          "properties": {
            "name": {
              "type": "string",
              "description": "Display name from the portal brand or plugin manifest."
            },
            "bg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Background CSS color from the portal brand, when known."
            },
            "fg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Foreground CSS color from the portal brand, when known."
            },
            "accent": {
              "type": [
                "string",
                "null"
              ],
              "description": "Accent CSS color from the portal brand, when known."
            },
            "logo": {
              "type": [
                "string",
                "null"
              ],
              "description": "Logo URL or host-served embed path."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "limits": {
      "type": "object",
      "description": "Host-capped resource defaults (workerd budgets + shared disk). Always set.",
      "required": [
        "cpu_ms",
        "subrequests",
        "max_cpu_ms",
        "max_subrequests",
        "disk_mib",
        "max_disk_mib",
        "memory_mib",
        "max_memory_mib",
        "cpu_cores",
        "max_cpu_cores",
        "extra_processes",
        "max_extra_processes",
        "known_bindings"
      ],
      "properties": {
        "cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd subrequest budget.",
          "minimum": 0
        },
        "max_cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd CPU milliseconds.",
          "minimum": 0
        },
        "max_subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd subrequests.",
          "minimum": 0
        },
        "disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "max_disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for plugin state disk in mebibytes.",
          "minimum": 0
        },
        "memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default jail memory in mebibytes.",
          "minimum": 0
        },
        "max_memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for jail memory in mebibytes.",
          "minimum": 0
        },
        "cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Manifest / default jail CPU in cores (2 d.p.)."
        },
        "max_cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Host max jail CPU in cores (2 d.p.)."
        },
        "jail_cpu_cores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Optional `[plugins.jail]` ceiling in cores."
        },
        "extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Default extra process/thread budget (native consent).",
          "minimum": 0
        },
        "max_extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for extra process budget.",
          "minimum": 0
        },
        "known_bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Suggested / default host bindings operators may grant."
        }
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

Status `403`: The caller's role does not allow this route.

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

Status `404`: No such plugin.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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

Status `500`: The grant could not be read.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
string
```

Schema:

A `text/plain` body: a string.

## `POST /api/plugins/{id}/consent`

Persists an approved plugin grant after validating operator overrides.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `id` (path) *string*, **required** — Plugin id.
- `plugin_key` (query) *string* — Canonical PluginKey; when set, wins over the path alias.

### Request body

`application/json`: [`PluginConsentApproveRequest`](schemas.md#pluginconsentapproverequest)

- `approve` *boolean*, **required** — Must be true; a false value is rejected with 400.
- `grant` *[`PluginGrantOverride`](schemas.md#plugingrantoverride) or null* — Optional operator overrides applied on top of the manifest request.

### Responses

- `200` — The grant was stored. Body: [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`)
  - `plugin_id` *string*, **required** — Plugin id whose grant is being reviewed.
  - `plugin_key` *string*, **required** — Canonical PluginKey for this install (empty when unknown).
  - `provenance` *string*, **required** — Host-evaluated provenance label.
  - `runtime` *string*, **required** — Guest runtime (`native` or `workerd`) — drives which controls are enforceable.
  - `request` *[`PluginGrantView`](schemas.md#plugingrantview)*, **required** — Manifest-derived grant the plugin is asking for.
  - `covered` *boolean*, **required** — True when an existing on-disk grant already satisfies the request.
  - `summary` *array of string*, **required** — Operator-facing bullet list of requested capabilities.
  - `existing` *[`PluginGrantView`](schemas.md#plugingrantview) or null* — Previously approved grant, when one is stored.
  - `brand` *[`PluginConsentBrand`](schemas.md#pluginconsentbrand) or null* — Optional portal or manifest branding for the dialog.
  - `limits` *[`PluginConsentLimits`](schemas.md#pluginconsentlimits)*, **required** — Host-capped resource defaults (workerd budgets + shared disk). Always set.
- `400` — An override is not allowed. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — No such plugin. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `500` — The grant could not be stored. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<id>/consent' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"approve": true}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<id>/consent' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"approve": true}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/plugins/<id>/consent",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"approve": True},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<id>/consent", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"approve": true}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/plugins/<id>/consent")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"approve": true}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The grant was stored.

```json
{
  "plugin_id": "string",
  "plugin_key": "string",
  "provenance": "string",
  "runtime": "string",
  "request": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "covered": true,
  "summary": [
    "string"
  ],
  "existing": {
    "pluginKey": "string",
    "pluginId": "string",
    "entrypoints": [
      "string"
    ],
    "producers": [
      "string"
    ],
    "consumers": [
      {}
    ],
    "jobs": [
      "string"
    ],
    "networkMode": "string",
    "domains": [
      "string"
    ],
    "manifestDomains": [
      "string"
    ],
    "operatorAddedDomains": [
      "string"
    ],
    "operatorDeniedDomains": [
      "string"
    ],
    "tcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorAddedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "operatorDeniedTcp": [
      {
        "host": "string",
        "ports": []
      }
    ],
    "addressCidrs": [
      "string"
    ],
    "operatorAddedCidrs": [
      "string"
    ],
    "operatorDeniedCidrs": [
      "string"
    ],
    "allowUndeclaredPublicRedirects": true,
    "grantRevision": "string",
    "authorityRevision": "string",
    "bindings": [
      "string"
    ],
    "compatibilityFlags": [
      "string"
    ],
    "approvedAt": "string",
    "cpuMs": 0,
    "subrequests": 0,
    "diskMib": 0,
    "memoryMib": 0,
    "cpuCores": 0,
    "extraProcesses": 0
  },
  "brand": {
    "name": "string",
    "bg": "string",
    "fg": "string",
    "accent": "string",
    "logo": "string"
  },
  "limits": {
    "cpu_ms": 0,
    "subrequests": 0,
    "max_cpu_ms": 0,
    "max_subrequests": 0,
    "disk_mib": 0,
    "max_disk_mib": 0,
    "memory_mib": 0,
    "max_memory_mib": 0,
    "cpu_cores": 0,
    "max_cpu_cores": 0,
    "jail_cpu_cores": 0,
    "extra_processes": 0,
    "max_extra_processes": 0,
    "known_bindings": [
      "string"
    ]
  }
}
```

Schema:

```json
{
  "type": "object",
  "description": "Consent dialog payload for one plugin.",
  "required": [
    "plugin_id",
    "plugin_key",
    "provenance",
    "runtime",
    "request",
    "covered",
    "summary",
    "limits"
  ],
  "properties": {
    "plugin_id": {
      "type": "string",
      "description": "Plugin id whose grant is being reviewed."
    },
    "plugin_key": {
      "type": "string",
      "description": "Canonical PluginKey for this install (empty when unknown)."
    },
    "provenance": {
      "type": "string",
      "description": "Host-evaluated provenance label."
    },
    "runtime": {
      "type": "string",
      "description": "Guest runtime (`native` or `workerd`) — drives which controls are enforceable."
    },
    "request": {
      "type": "object",
      "description": "Manifest-derived grant the plugin is asking for.",
      "required": [
        "pluginKey",
        "pluginId",
        "entrypoints",
        "producers",
        "consumers",
        "jobs",
        "networkMode",
        "domains",
        "manifestDomains",
        "operatorAddedDomains",
        "operatorDeniedDomains",
        "tcp",
        "operatorAddedTcp",
        "operatorDeniedTcp",
        "addressCidrs",
        "operatorAddedCidrs",
        "operatorDeniedCidrs",
        "allowUndeclaredPublicRedirects",
        "grantRevision",
        "authorityRevision",
        "bindings",
        "compatibilityFlags",
        "approvedAt"
      ],
      "properties": {
        "pluginKey": {
          "type": "string",
          "description": "Canonical provenance-qualified PluginKey."
        },
        "pluginId": {
          "type": "string",
          "description": "Plugin id this grant applies to."
        },
        "entrypoints": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
        },
        "producers": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Event types the operator approved the plugin to publish."
        },
        "consumers": {
          "type": "array",
          "items": {
            "type": "object"
          },
          "description": "Approved event-consumer subscriptions."
        },
        "jobs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Approved job trigger types."
        },
        "networkMode": {
          "type": "string",
          "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
        },
        "domains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Hostnames the operator approved for egress."
        },
        "manifestDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Destinations requested by the current manifest."
        },
        "operatorAddedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added destinations beyond the manifest."
        },
        "operatorDeniedDomains": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied destinations still listed by the manifest."
        },
        "tcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Effective raw TCP grants."
        },
        "operatorAddedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-added TCP grants."
        },
        "operatorDeniedTcp": {
          "type": "array",
          "items": {
            "type": "object",
            "description": "Operator-facing TCP grant (`host` + `ports`).",
            "required": [
              "host",
              "ports"
            ],
            "properties": {
              "host": {
                "type": "string",
                "description": "Hostname or `*.` wildcard."
              },
              "ports": {
                "type": "array",
                "items": {
                  "type": "integer",
                  "format": "int32",
                  "minimum": 0
                },
                "description": "Destination ports."
              }
            }
          },
          "description": "Operator-denied TCP grants still listed by the manifest."
        },
        "addressCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Effective address-space CIDRs."
        },
        "operatorAddedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-added CIDRs."
        },
        "operatorDeniedCidrs": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Operator-denied CIDRs still listed by the manifest."
        },
        "allowUndeclaredPublicRedirects": {
          "type": "boolean",
          "description": "Undeclared public redirect permission."
        },
        "grantRevision": {
          "type": "string",
          "description": "SHA-256 of persisted operator consent (no host overlays)."
        },
        "authorityRevision": {
          "type": "string",
          "description": "SHA-256 of the canonical effective authority."
        },
        "bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Host bindings the operator approved."
        },
        "compatibilityFlags": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Extra compatibility flags stored on the grant."
        },
        "approvedAt": {
          "type": "string",
          "description": "RFC 3339 timestamp when the grant was last approved."
        },
        "cpuMs": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional workerd subrequest budget.",
          "minimum": 0
        },
        "diskMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "memoryMib": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Optional jail memory in mebibytes.",
          "minimum": 0
        },
        "cpuCores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Jail CPU in cores (2 d.p.); absent for workerd."
        },
        "extraProcesses": {
          "type": [
            "integer",
            "null"
          ],
          "format": "int32",
          "description": "Extra process/thread budget beyond launcher overhead; native only.",
          "minimum": 0
        }
      }
    },
    "covered": {
      "type": "boolean",
      "description": "True when an existing on-disk grant already satisfies the request."
    },
    "summary": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Operator-facing bullet list of requested capabilities."
    },
    "existing": {
      "oneOf": [
        {
          "type": "object",
          "description": "Previously approved grant, when one is stored.",
          "required": [
            "pluginKey",
            "pluginId",
            "entrypoints",
            "producers",
            "consumers",
            "jobs",
            "networkMode",
            "domains",
            "manifestDomains",
            "operatorAddedDomains",
            "operatorDeniedDomains",
            "tcp",
            "operatorAddedTcp",
            "operatorDeniedTcp",
            "addressCidrs",
            "operatorAddedCidrs",
            "operatorDeniedCidrs",
            "allowUndeclaredPublicRedirects",
            "grantRevision",
            "authorityRevision",
            "bindings",
            "compatibilityFlags",
            "approvedAt"
          ],
          "properties": {
            "pluginKey": {
              "type": "string",
              "description": "Canonical provenance-qualified PluginKey."
            },
            "pluginId": {
              "type": "string",
              "description": "Plugin id this grant applies to."
            },
            "entrypoints": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Exported entrypoints the operator approved (`storefront`, `cli`, …)."
            },
            "producers": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Event types the operator approved the plugin to publish."
            },
            "consumers": {
              "type": "array",
              "items": {
                "type": "object"
              },
              "description": "Approved event-consumer subscriptions."
            },
            "jobs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Approved job trigger types."
            },
            "networkMode": {
              "type": "string",
              "description": "Approved egress mode (`deny`, `allowlist`, and similar)."
            },
            "domains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Hostnames the operator approved for egress."
            },
            "manifestDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Destinations requested by the current manifest."
            },
            "operatorAddedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added destinations beyond the manifest."
            },
            "operatorDeniedDomains": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied destinations still listed by the manifest."
            },
            "tcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Effective raw TCP grants."
            },
            "operatorAddedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-added TCP grants."
            },
            "operatorDeniedTcp": {
              "type": "array",
              "items": {
                "type": "object",
                "description": "Operator-facing TCP grant (`host` + `ports`).",
                "required": [
                  "host",
                  "ports"
                ],
                "properties": {
                  "host": {
                    "type": "string",
                    "description": "Hostname or `*.` wildcard."
                  },
                  "ports": {
                    "type": "array",
                    "items": {
                      "type": "integer",
                      "format": "int32",
                      "minimum": 0
                    },
                    "description": "Destination ports."
                  }
                }
              },
              "description": "Operator-denied TCP grants still listed by the manifest."
            },
            "addressCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Effective address-space CIDRs."
            },
            "operatorAddedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-added CIDRs."
            },
            "operatorDeniedCidrs": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Operator-denied CIDRs still listed by the manifest."
            },
            "allowUndeclaredPublicRedirects": {
              "type": "boolean",
              "description": "Undeclared public redirect permission."
            },
            "grantRevision": {
              "type": "string",
              "description": "SHA-256 of persisted operator consent (no host overlays)."
            },
            "authorityRevision": {
              "type": "string",
              "description": "SHA-256 of the canonical effective authority."
            },
            "bindings": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Host bindings the operator approved."
            },
            "compatibilityFlags": {
              "type": "array",
              "items": {
                "type": "string"
              },
              "description": "Extra compatibility flags stored on the grant."
            },
            "approvedAt": {
              "type": "string",
              "description": "RFC 3339 timestamp when the grant was last approved."
            },
            "cpuMs": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd CPU budget in milliseconds.",
              "minimum": 0
            },
            "subrequests": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional workerd subrequest budget.",
              "minimum": 0
            },
            "diskMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional plugin state disk budget in mebibytes.",
              "minimum": 0
            },
            "memoryMib": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Optional jail memory in mebibytes.",
              "minimum": 0
            },
            "cpuCores": {
              "type": [
                "number",
                "null"
              ],
              "format": "double",
              "description": "Jail CPU in cores (2 d.p.); absent for workerd."
            },
            "extraProcesses": {
              "type": [
                "integer",
                "null"
              ],
              "format": "int32",
              "description": "Extra process/thread budget beyond launcher overhead; native only.",
              "minimum": 0
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "brand": {
      "oneOf": [
        {
          "type": "object",
          "description": "Optional portal or manifest branding for the dialog.",
          "required": [
            "name"
          ],
          "properties": {
            "name": {
              "type": "string",
              "description": "Display name from the portal brand or plugin manifest."
            },
            "bg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Background CSS color from the portal brand, when known."
            },
            "fg": {
              "type": [
                "string",
                "null"
              ],
              "description": "Foreground CSS color from the portal brand, when known."
            },
            "accent": {
              "type": [
                "string",
                "null"
              ],
              "description": "Accent CSS color from the portal brand, when known."
            },
            "logo": {
              "type": [
                "string",
                "null"
              ],
              "description": "Logo URL or host-served embed path."
            }
          }
        },
        {
          "type": "null"
        }
      ]
    },
    "limits": {
      "type": "object",
      "description": "Host-capped resource defaults (workerd budgets + shared disk). Always set.",
      "required": [
        "cpu_ms",
        "subrequests",
        "max_cpu_ms",
        "max_subrequests",
        "disk_mib",
        "max_disk_mib",
        "memory_mib",
        "max_memory_mib",
        "cpu_cores",
        "max_cpu_cores",
        "extra_processes",
        "max_extra_processes",
        "known_bindings"
      ],
      "properties": {
        "cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd CPU budget in milliseconds.",
          "minimum": 0
        },
        "subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Requested workerd subrequest budget.",
          "minimum": 0
        },
        "max_cpu_ms": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd CPU milliseconds.",
          "minimum": 0
        },
        "max_subrequests": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for workerd subrequests.",
          "minimum": 0
        },
        "disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default plugin state disk budget in mebibytes.",
          "minimum": 0
        },
        "max_disk_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for plugin state disk in mebibytes.",
          "minimum": 0
        },
        "memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Default jail memory in mebibytes.",
          "minimum": 0
        },
        "max_memory_mib": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for jail memory in mebibytes.",
          "minimum": 0
        },
        "cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Manifest / default jail CPU in cores (2 d.p.)."
        },
        "max_cpu_cores": {
          "type": "number",
          "format": "double",
          "description": "Host max jail CPU in cores (2 d.p.)."
        },
        "jail_cpu_cores": {
          "type": [
            "number",
            "null"
          ],
          "format": "double",
          "description": "Optional `[plugins.jail]` ceiling in cores."
        },
        "extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Default extra process/thread budget (native consent).",
          "minimum": 0
        },
        "max_extra_processes": {
          "type": "integer",
          "format": "int32",
          "description": "Host hard cap for extra process budget.",
          "minimum": 0
        },
        "known_bindings": {
          "type": "array",
          "items": {
            "type": "string"
          },
          "description": "Suggested / default host bindings operators may grant."
        }
      }
    }
  }
}
```

Status `400`: An override is not allowed.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

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

Status `404`: No such plugin.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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

Status `500`: The grant could not be stored.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
string
```

Schema:

A `text/plain` body: a string.

## `GET /api/plugins/{kind}/{id}/logo`

Serve an embedded `plugin.toml` `logo` file from under the plugin install root.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

- `kind` (path) *string*, **required** — Plugin kind (`source`, `destination`, `integration`, …).
- `id` (path) *string*, **required** — Plugin id.

### Responses

- `200` — The logo (PNG, or SVG served with a restrictive CSP). Body: [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/svg+xml`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
  - `error` *string*, **required** — Stable machine slug.
  - `message` *string* — Explanation for people.
  - `status` *integer* — The HTTP status, repeated.
- `404` — The plugin has no logo. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
  - `error` *string*, **required** — Machine slug (same as [`ErrorBody::code`]).
  - `message` *string*, **required** — Operator-facing explanation.
  - `status` *integer (int32)*, **required** — Numeric HTTP status (duplicates the response status for clients). At least 0.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/plugins/<kind>/<id>/logo")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The logo (PNG, or SVG served with a restrictive CSP). The body is binary (`image/png`).

Schema:

A `image/png` body: binary bytes.

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

Status `403`: The caller's role does not allow this route.

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

Status `404`: The plugin has no logo.

```json
{
  "error": "not_found",
  "message": "That page or resource was not found.",
  "status": 404
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
string
```

Schema:

A `text/plain` body: a string.
