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

## `PATCH /api/settings`

Validates, writes, and reloads allowlisted Settings keys under `reload_lock`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PatchSettingsRequest`](schemas.md#patchsettingsrequest)

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

## `POST /api/database/migrate`

Copies library rows between database plugins and optionally switches `[database].plugin`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`DatabaseMigrateRequest`](schemas.md#databasemigraterequest)

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

## `PATCH /api/preferences`

Merges a preferences PATCH onto the caller’s stored row and writes it back.

**Access:** The operator, or any library session.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PatchPreferencesBody`](schemas.md#patchpreferencesbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The preferences after the change. | [`PreferencesResponse`](schemas.md#preferencesresponse) (`application/json`) |
| 400 | A value is invalid. | string (`text/plain`) |
| 401 | No signed-in caller. | string (`text/plain`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The request could not be read or written. | string (`text/plain`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
