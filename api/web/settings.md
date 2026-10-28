# Settings

Daemon settings and user preferences.

Back to the [web API reference](README.md).

## `GET /api/settings`

Returns file-backed and runtime-effective Settings plus per-plugin groups.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

- `200` — The settings. Body: [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`)
  - `settings` *map of string to string*, **required** — Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document.
  - `effective` *map of string to string*, **required** — Runtime-effective values after the last successful reload (auth, plugins).
  - `plugins` *array of [`PluginSettingsGroup`](schemas.md#pluginsettingsgroup)*, **required** — Per-plugin setting groups rendered by the Settings UI.
  - `host_cpu_cores_max` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.; equals logical CPU count).
  - `jail_cpu_cores` *number (double) or null* — Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).
  - `output_revision` *integer (int64) or null* — Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup.
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The settings could not be read. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/settings' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/settings",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/settings", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/settings")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings.

```json
{
  "settings": {
    "key": "string"
  },
  "effective": {
    "key": "string"
  },
  "plugins": [
    {
      "id": "string",
      "family": "string",
      "plugin_key": "string",
      "provenance": "string",
      "logo": "string",
      "settings": [
        {
          "key": "string",
          "label": "string",
          "value": "string",
          "value_type": "string"
        }
      ],
      "plugin_instance_id": "string",
      "display_name": "string",
      "is_default": true,
      "config_revision": 0,
      "deployment": {
        "deployment_id": "string",
        "node_id": "string",
        "desired": "present",
        "revision": 0,
        "status": "string",
        "detail": "string",
        "applied_config_revision": 0,
        "observed_at": "string"
      },
      "placement": {
        "replicas": 0,
        "constraints": [],
        "revision": 0,
        "placed": 0,
        "detail": "string",
        "scheduled_at": "string",
        "updated_at": "string"
      },
      "present_on": [
        "string"
      ]
    }
  ],
  "host_cpu_cores_max": 0,
  "jail_cpu_cores": 0,
  "output_revision": 0
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

Status `500`: The settings could not be read.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PATCH /api/settings`

Validates, writes, and reloads allowlisted Settings keys under `reload_lock`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PatchSettingsRequest`](schemas.md#patchsettingsrequest)

- `settings` *array of [`SettingsUpdate`](schemas.md#settingsupdate)*, **required** — Key/value pairs to validate, persist, and reload.
- `expected_output_revision` *integer (int64) or null* — `core.output` document revision the client read (`output_revision` in `GET /api/settings`). When set and stale, an `output.*` update is rejected with 409.

### Responses

- `200` — The settings after the change. Body: [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`)
  - `settings` *map of string to string*, **required** — Values as written in config.toml (and env overlays), plus `output.*` from the `core.output` database document.
  - `effective` *map of string to string*, **required** — Runtime-effective values after the last successful reload (auth, plugins).
  - `plugins` *array of [`PluginSettingsGroup`](schemas.md#pluginsettingsgroup)*, **required** — Per-plugin setting groups rendered by the Settings UI.
  - `host_cpu_cores_max` *number (double)*, **required** — Host max jail CPU in cores (2 d.p.; equals logical CPU count).
  - `jail_cpu_cores` *number (double) or null* — Optional global per-jail CPU ceiling in cores (from `[plugins.jail]`).
  - `output_revision` *integer (int64) or null* — Revision of the `core.output` document the `output.*` values in `settings` were read from; send it back as `expected_output_revision`. `effective["output.revision"]` is the revision this host has applied, which lags the document until the host's next wakeup.
- `400` — A key is unknown or a value is invalid. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — `output.*` keys were sent with a stale `expected_output_revision`, or another write won; the body's `current_revision` is the revision to retry against (`revision_conflict`). Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The settings could not be written. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"settings": [{"key": "string", "value": "string"}]}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/settings' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"settings": [{"key": "string", "value": "string"}]}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/settings",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"settings": [{"key": "string", "value": "string"}]},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/settings", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"settings": [{"key": "string", "value": "string"}]}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/settings")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"settings": [{"key": "string", "value": "string"}]}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The settings after the change.

```json
{
  "settings": {
    "key": "string"
  },
  "effective": {
    "key": "string"
  },
  "plugins": [
    {
      "id": "string",
      "family": "string",
      "plugin_key": "string",
      "provenance": "string",
      "logo": "string",
      "settings": [
        {
          "key": "string",
          "label": "string",
          "value": "string",
          "value_type": "string"
        }
      ],
      "plugin_instance_id": "string",
      "display_name": "string",
      "is_default": true,
      "config_revision": 0,
      "deployment": {
        "deployment_id": "string",
        "node_id": "string",
        "desired": "present",
        "revision": 0,
        "status": "string",
        "detail": "string",
        "applied_config_revision": 0,
        "observed_at": "string"
      },
      "placement": {
        "replicas": 0,
        "constraints": [],
        "revision": 0,
        "placed": 0,
        "detail": "string",
        "scheduled_at": "string",
        "updated_at": "string"
      },
      "present_on": [
        "string"
      ]
    }
  ],
  "host_cpu_cores_max": 0,
  "jail_cpu_cores": 0,
  "output_revision": 0
}
```

Status `400`: A key is unknown or a value is invalid.

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

Status `409`: `output.*` keys were sent with a stale `expected_output_revision`, or another write won; the body's `current_revision` is the revision to retry against (`revision_conflict`).

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The settings could not be written.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/database/migrate`

Copies library rows between database plugins and optionally switches `[database].plugin`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`DatabaseMigrateRequest`](schemas.md#databasemigraterequest)

- `from` *string or null* — Source plugin id. Defaults to the active `[database].plugin` before reload.
- `to` *string*, **required** — Destination database plugin id to copy into.
- `dry_run` *boolean* — When true, count rows without writing.
- `force` *boolean* — When true, overwrite a destination that is not empty.
- `apply` *boolean* — Update `[database].plugin` in config.toml after a successful copy.

### Responses

- `200` — The rows were copied. Body: [`ActionResponse`](schemas.md#actionresponse) (`application/json`)
  - `ok` *boolean*, **required** — True when the action was accepted or completed.
  - `message` *string*, **required** — Operator-facing status text.
  - `job_id` *string*, **required** — Queued job id, or empty when the action is synchronous.
  - `error` *string or null* — Machine slug when admission failed (`conflict`, `queue_full`).
- `400` — The target is invalid. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `401` — The caller is not signed in, or the credential is wrong. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `403` — The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `409` — The migration target matches more than one database plugin. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The copy failed. Body: [`ErrorJson`](schemas.md#errorjson) (`application/json`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/database/migrate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"to": "string"}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/database/migrate' `
    -Method Post `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{"to": "string"}'
```

```python
import requests

response = requests.post(
    "http://127.0.0.1:8787/api/database/migrate",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={"to": "string"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/database/migrate", {
  method: "POST",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({"to": "string"}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .post("http://127.0.0.1:8787/api/database/migrate")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({"to": "string"}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The rows were copied.

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

Status `400`: The target is invalid.

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

Status `409`: The migration target matches more than one database plugin.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The copy failed.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/preferences`

Loads preferences for the caller with a 3s timeout, creating defaults when missing.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

- `200` — The caller's preferences. Body: [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`)
  - `default_view` *string*, **required** — Landing view id (`library`, `discover`, …) after normalization.
  - `disabled_shelves` *array of string*, **required** — Lowercased Discover shelf ids the user has hidden.
  - `discover_sort` *string*, **required** — Discover sort key (`relevance`, `popularity`, …).
  - `discover_sort_dir` *string*, **required** — Discover sort direction (`asc` or `desc`).
  - `discover_language` *string or null* — `null` = use browser language on the client.
  - `discover_excluded_sources` *array of string*, **required** — Lowercased store ids hidden from Discover results.
  - `theme` *string*, **required** — Appearance preference (`system`, `light`, or `dark`).
- `401` — No signed-in caller. Body: string (`text/plain`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -H 'Authorization: Bearer <operatorToken>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/preferences' `
    -Headers @{ Authorization = 'Bearer <operatorToken>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/preferences",
    headers={"Authorization": "Bearer <operatorToken>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/preferences", {
  headers: { Authorization: "Bearer <operatorToken>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/preferences")
    .header("Authorization", "Bearer <operatorToken>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The caller's preferences.

```json
{
  "default_view": "string",
  "disabled_shelves": [
    "string"
  ],
  "discover_sort": "string",
  "discover_sort_dir": "string",
  "discover_language": "string",
  "discover_excluded_sources": [
    "string"
  ],
  "theme": "string"
}
```

Status `401`: No signed-in caller. The body is `text/plain`.

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `PATCH /api/preferences`

Merges a preferences PATCH onto the caller’s stored row and writes it back.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchPreferencesBody`](schemas.md#patchpreferencesbody)

- `default_view` *string or null* — Replacement landing view when set.
- `disabled_shelves` *array of string or null* — Replacement hidden-shelf list when set.
- `discover_sort` *string or null* — Replacement Discover sort key when set.
- `discover_sort_dir` *string or null* — Replacement Discover sort direction when set.
- `discover_language` *string or null* — Present + value sets language; present + `null` clears to browser default; omitted leaves unchanged.
- `discover_excluded_sources` *array of string or null* — Replacement excluded-store list when set.
- `theme` *string or null* — Replacement appearance preference when set (`system`, `light`, `dark`).

### Responses

- `200` — The preferences after the change. Body: [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`)
  - `default_view` *string*, **required** — Landing view id (`library`, `discover`, …) after normalization.
  - `disabled_shelves` *array of string*, **required** — Lowercased Discover shelf ids the user has hidden.
  - `discover_sort` *string*, **required** — Discover sort key (`relevance`, `popularity`, …).
  - `discover_sort_dir` *string*, **required** — Discover sort direction (`asc` or `desc`).
  - `discover_language` *string or null* — `null` = use browser language on the client.
  - `discover_excluded_sources` *array of string*, **required** — Lowercased store ids hidden from Discover results.
  - `theme` *string*, **required** — Appearance preference (`system`, `light`, or `dark`).
- `400` — A value is invalid. Body: string (`text/plain`)
- `401` — No signed-in caller. Body: string (`text/plain`)
- `403` — A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. Body: [`ApiError`](schemas.md#apierror) (`application/json`)
- `500` — The request could not be read or written. Body: string (`text/plain`)
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/preferences' `
    -Method Patch `
    -Headers @{ Authorization = 'Bearer <operatorToken>' } `
    -ContentType 'application/json' `
    -Body '{}'
```

```python
import requests

response = requests.patch(
    "http://127.0.0.1:8787/api/preferences",
    headers={"Authorization": "Bearer <operatorToken>"},
    json={},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/preferences", {
  method: "PATCH",
  headers: { Authorization: "Bearer <operatorToken>", "Content-Type": "application/json" },
  body: JSON.stringify({}),
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .patch("http://127.0.0.1:8787/api/preferences")
    .header("Authorization", "Bearer <operatorToken>")
    .json(&serde_json::json!({}))
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The preferences after the change.

```json
{
  "default_view": "string",
  "disabled_shelves": [
    "string"
  ],
  "discover_sort": "string",
  "discover_sort_dir": "string",
  "discover_language": "string",
  "discover_excluded_sources": [
    "string"
  ],
  "theme": "string"
}
```

Status `400`: A value is invalid. The body is `text/plain`.

Status `401`: No signed-in caller. The body is `text/plain`.

Status `403`: A write sent with a session cookie but without a matching `Origin` or `Referer` is refused.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `500`: The request could not be read or written. The body is `text/plain`.

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
