# Plugins

Plugin packages, instances, consent and deployment.

Back to the [web API reference](README.md).

## `GET /api/config/plugin-instances`

`GET /api/config/plugin-instances`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Every plugin instance with its deployments. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/config/plugin-instances`

`POST /api/config/plugin-instances`: a second (non-default) instance of
a plugin, with an empty document so schema defaults apply. With
`node_id`, the instance is local to that node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CreatePluginInstance`](schemas.md#createplugininstance)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `plugin_key` | string | yes | Canonical plugin key; the plugin must have a pinned schema. |
| `display_name` | string | no | Operator-facing name. |
| `node_id` | string or null | no | Make the instance local to this node (H-2): it deploys only there and is never placed by the scheduler. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new instance. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance id is taken. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/config/plugin-instances/{id}`

`DELETE /api/config/plugin-instances/{id}`: removes an instance that has
no deployment left.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance was removed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance is a default or still deployed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/config/plugin-instances/{id}/config`

`GET /api/config/plugin-instances/{id}/config`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance's settings document (secrets as presence flags). | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/plugin-instances/{id}/config`

`PUT /api/config/plugin-instances/{id}/config`.

Operator principal only. The route is mounted on the operator router.
The document is validated against the plugin's pinned schema, so this
node does not need the guest installed.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Request body

`application/json`: [`PutPluginInstanceConfig`](schemas.md#putplugininstanceconfig)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expected_revision` | integer (int64) | yes | Revision the caller last observed. |
| `settings` | object | no | Scalar settings. Secret values are not accepted here. |
| `secret_refs` | array of object | no | Secret refs to keep. Omitted refs are dropped from the document; the ciphertext stays sealed until the secret is overwritten. |
| `secrets` | map of string to string | no | Secret values to seal, keyed by the declared `secret` setting. Each is sealed into `encrypted_secrets` and referenced from the document; the plaintext is never stored in the document or echoed back. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The document after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
Deployed switch. Creates the row when the node has none.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |
| `node_id` | path | string | yes | Node id. |

### Request body

`application/json`: [`PutPluginDeployment`](schemas.md#putplugindeployment)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `desired` | [`DesiredState`](schemas.md#desiredstate) | yes | What the node should do: `present` or `absent` (the scheduler's `placed` / `unplaced` are accepted too). |
| `expected_revision` | integer (int64) or null (int64) | no | Revision the caller last observed; required when the row exists and the caller wants compare-and-swap. Omitted means "replace whatever is current". |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The deployment after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance is placed by the scheduler. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/plugin-instances/{id}/placement`

`PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
deployments to the scheduler with a replica count and constraints.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Request body

`application/json`: [`PutPluginPlacement`](schemas.md#putpluginplacement)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `replicas` | integer or `all` | yes | A replica count (0 stops every replica) or `"all"`. |
| `constraints` | array of string | no | Label constraints (`name`, `name=value`, `!name`). |
| `expected_revision` | integer (int64) or null (int64) | no | Current revision to replace; omit (or 0) to create. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The scheduler now places the instance. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/config/plugin-instances/{id}/placement`

`DELETE /api/config/plugin-instances/{id}/placement`: return the
instance to hand-managed deployments; its deployments stay as they
are.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance is back to manual deployment. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such instance.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/config/plugins`

`GET /api/config/plugins`: every installed plugin.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Every installed plugin. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/config/plugins`

`POST /api/config/plugins`: install (and maybe activate) a version.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PostPluginInstall`](schemas.md#postplugininstall)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `coordinate` | string or null | no | Registry coordinate (`cargo:…@v`, `registry:…#name@v`). |
| `package` | string or null | no | Operator-placed package directory under `plugin-packages/`. |
| `activate` | boolean | no | Make the new version active even when another is. |
| `approve` | boolean | no | Record the requested consent as approved. |
| `allow_unverified_publisher` | boolean | no | Accept a publisher without independent authenticity. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The installed version. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | A local package is missing its `package.json` or `archive.tar.gz`. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The version is already installed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The package catalog could not be reached or answered badly. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: A local package is missing its `package.json` or `archive.tar.gz`.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The configuration could not be read or written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/config/plugins/{key}`

`GET /api/config/plugins/{key}`: one installed plugin with its pinned
settings schema.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The plugin with its pinned versions. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `DELETE /api/config/plugins/{key}`

`DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
cleared, placements dropped, every deployment `absent`).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The plugin was uninstalled. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PUT /api/config/plugins/{key}/active`

`PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Request body

`application/json`: [`PutActiveVersion`](schemas.md#putactiveversion)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `version` | string | yes | Installed version to activate. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The active version after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/config/plugins/{key}/versions/{version}/approve`

`POST /api/config/plugins/{key}/versions/{version}/approve`: record the
version's requested consent as approved for every node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |
| `version` | path | string | yes | Semver version. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The version's grant was approved. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/plugins/{id}/consent`

Loads the consent dialog for one plugin, including coverage against the stored grant.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin id. |
| `plugin_key` | query | string | no | Canonical PluginKey; when set, wins over the path alias. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | What the plugin asks for and what is granted. | [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The grant could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `404`: No such plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The grant could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/plugins/{id}/consent`

Persists an approved plugin grant after validating operator overrides.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin id. |
| `plugin_key` | query | string | no | Canonical PluginKey; when set, wins over the path alias. |

### Request body

`application/json`: [`PluginConsentApproveRequest`](schemas.md#pluginconsentapproverequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `approve` | boolean | yes | Must be true; a false value is rejected with 400. |
| `grant` | [`PluginGrantOverride`](schemas.md#plugingrantoverride) or null | no | Optional operator overrides applied on top of the manifest request. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The grant was stored. | [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`) |
| 400 | An override is not allowed. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The grant could not be stored. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `400`: An override is not allowed.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `403`: The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `500`: The grant could not be stored.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/plugins/{kind}/{id}/logo`

Serve an embedded `plugin.toml` `logo` file from under the plugin install root.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `kind` | path | string | yes | Plugin kind (`source`, `destination`, `integration`, …). |
| `id` | path | string | yes | Plugin id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The logo (PNG, or SVG served with a restrictive CSP). | [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/svg+xml`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The plugin has no logo. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

Status `200`: The logo (PNG, or SVG served with a restrictive CSP). The body is `image/png`, `image/svg+xml`.

Status `401`: The caller is not signed in, or the credential is wrong.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
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

Status `404`: The plugin has no logo.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
