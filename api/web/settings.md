# Settings

Daemon settings and user preferences.

Back to the [web API reference](README.md).

## `GET /api/settings`

Returns file-backed and runtime-effective Settings plus per-plugin groups.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings. | [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The settings could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`:

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

## `PATCH /api/settings`

Validates, writes, and reloads allowlisted Settings keys under `reload_lock`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PatchSettingsRequest`](schemas.md#patchsettingsrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `settings` | array of [`SettingsUpdate`](schemas.md#settingsupdate) | yes | Key/value pairs to validate, persist, and reload. |
| `expected_output_revision` | integer (int64) or null (int64) | no | `core.output` document revision the client read (`output_revision` in `GET /api/settings`). When set and stale, an `output.*` update is rejected with 409. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings after the change. | [`SettingsResponse`](schemas.md#settingsresponse) (`application/json`) |
| 400 | A key is unknown or a value is invalid. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | `output.*` keys were sent with a stale `expected_output_revision`, or another write won; the body's `current_revision` is the revision to retry against (`revision_conflict`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The settings could not be written. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/settings' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"settings": [{"key": "string", "value": "string"}]}'
```

### Example response

Status `200`:

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

## `POST /api/database/migrate`

Copies library rows between database plugins and optionally switches `[database].plugin`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`DatabaseMigrateRequest`](schemas.md#databasemigraterequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string or null | no | Source plugin id. Defaults to the active `[database].plugin` before reload. |
| `to` | string | yes | Destination database plugin id to copy into. |
| `dry_run` | boolean | no | When true, count rows without writing. |
| `force` | boolean | no | When true, overwrite a destination that is not empty. |
| `apply` | boolean | no | Update `[database].plugin` in config.toml after a successful copy. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The rows were copied. | [`ActionResponse`](schemas.md#actionresponse) (`application/json`) |
| 400 | The target is invalid. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The migration target matches more than one database plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The copy failed. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/database/migrate' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"to": "string"}'
```

### Example response

Status `200`:

```json
{
  "ok": true,
  "message": "string",
  "job_id": "string",
  "error": "string"
}
```

## `GET /api/preferences`

Loads preferences for the caller with a 3s timeout, creating defaults when missing.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The caller's preferences. | [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `200`:

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

## `PATCH /api/preferences`

Merges a preferences PATCH onto the caller’s stored row and writes it back.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchPreferencesBody`](schemas.md#patchpreferencesbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `default_view` | string or null | no | Replacement landing view when set. |
| `disabled_shelves` | array of string or null | no | Replacement hidden-shelf list when set. |
| `discover_sort` | string or null | no | Replacement Discover sort key when set. |
| `discover_sort_dir` | string or null | no | Replacement Discover sort direction when set. |
| `discover_language` | string or null | no | Present + value sets language; present + `null` clears to browser default; omitted leaves unchanged. |
| `discover_excluded_sources` | array of string or null | no | Replacement excluded-store list when set. |
| `theme` | string or null | no | Replacement appearance preference when set (`system`, `light`, `dark`). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The preferences after the change. | [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`) |
| 400 | A value is invalid. | string (`text/plain`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/preferences' \
    -X PATCH \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`:

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
