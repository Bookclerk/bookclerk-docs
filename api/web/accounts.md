# Accounts

Store accounts a library user connects (the Accounts page).

Back to the [web API reference](README.md).

## `POST /api/portal/redeem`

Redeems a claim ticket, optionally setting a first password, and sets the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`RedeemBody`](schemas.md#redeembody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ticket` | string | yes | Raw claim-ticket token to redeem into a portal session. |
| `nonce` | string | yes | Browser-generated one-time nonce persisted across HTTP retries. |
| `password` | string or null | no | Required when the linked local user has no password (invite / reset). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in; the response sets the portal session cookie. | any JSON (`application/json`) |
| 400 | The ticket is invalid, used or expired. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/redeem' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"ticket": "string", "nonce": "string"}'
```

### Example response

Status `200`: Signed in; the response sets the portal session cookie. The body is JSON the schemas do not describe yet.

Status `400`: The ticket is invalid, used or expired.

```json
{
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/login/integration`

Authenticates against an enabled integration and mints a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`IntegrationLoginBody`](schemas.md#integrationloginbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | string | yes | Integration plugin id (must be enabled in config). |
| `username` | string | yes | Provider login name forwarded to `authenticate_user`. |
| `password` | string | yes | Provider password; never logged. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in through the integration. | any JSON (`application/json`) |
| 400 | The request is invalid, or the integration refused the login. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/login/integration' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"provider": "string", "username": "string", "password": "string"}'
```

### Example response

Status `200`: Signed in through the integration. The body is JSON the schemas do not describe yet.

Status `400`: The request is invalid, or the integration refused the login.

```json
{
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/logout`

Revokes the portal session row and clears the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed out (`{"ok": true}`); the cookie is cleared. | any JSON (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/logout' \
    -X POST
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/portal/me`

Returns the signed-in portal identity, or 401 when the cookie is missing or expired.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The signed-in identity. | [`MeResponse`](schemas.md#meresponse) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/me' \
    -b 'bookclerk_portal_session=<portalSession>'
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

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/portal/sources`

Lists the deployed content sources (one per plugin, through its default
instance) with portal auth mode, config options, and brand colors.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Deployed content sources. | [`SourcesResponse`](schemas.md#sourcesresponse) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources'
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/sources/{id}/login`

Connects a password storefront, upserts the account, and links it to the signed-in identity.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Source plugin id. |

### Request body

`application/json`: [`PasswordLoginBody`](schemas.md#passwordloginbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | string | yes | Store account email / username. |
| `password` | string | yes | Store account password; never logged. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The account was connected. | any JSON (`application/json`) |
| 400 | The login failed. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<id>/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "string", "password": "string"}'
```

### Example response

Status `200`: The account was connected. The body is JSON the schemas do not describe yet.

Status `400`: The login failed.

```json
{
  "error": "string"
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/sources/{id}/oauth/start`

Starts interactive OAuth for an OAuth storefront and returns the browser login URL.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Source plugin id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The browser login URL. | any JSON (`application/json`) |
| 400 | The source does not use OAuth. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<id>/oauth/start' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

### Example response

Status `200`: The browser login URL. The body is JSON the schemas do not describe yet.

Status `400`: The source does not use OAuth.

```json
{
  "error": "string"
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/libro/login`

**Deprecated.**

Legacy `POST /libro/login` alias that forwards to the Libro password-login handler.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasswordLoginBody`](schemas.md#passwordloginbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | string | yes | Store account email / username. |
| `password` | string | yes | Store account password; never logged. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The account was connected. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/libro/login' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787' \
    -H 'Content-Type: application/json' \
    -d '{"email": "string", "password": "string"}'
```

### Example response

Status `200`: The account was connected. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/audible/start`

**Deprecated.**

Legacy `POST /audible/start` alias that forwards to Audible OAuth start.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The browser login URL. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/audible/start' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

### Example response

Status `200`: The browser login URL. The body is JSON the schemas do not describe yet.

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `GET /api/portal/connections`

Lists store accounts linked to the signed-in identity, including disabled sources.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Store accounts linked to the caller. | [`ConnectionsResponse`](schemas.md#connectionsresponse) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/connections' \
    -b 'bookclerk_portal_session=<portalSession>'
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

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.

## `POST /api/portal/sources/{source}/connections/{account_id}/revoke`

Unlinks a store account from this identity and deletes secrets when no other identity remains.

The account is addressed by `(source, account_id)`: the same store id may
exist under two plugins. `source` is the alias or plugin key of the link.

When the last link goes, the plugin first signs the account out at the
store (for example, Audible deregisters its device), best effort.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `source` | path | string | yes | Source plugin id. |
| `account_id` | path | string | yes | Store account id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The account was unlinked. | any JSON (`application/json`) |
| 400 | The account is not linked to this identity. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/portal/sources/<source>/connections/<account_id>/revoke' \
    -X POST \
    -b 'bookclerk_portal_session=<portalSession>' \
    -H 'Origin: http://127.0.0.1:8787'
```

### Example response

Status `200`: The account was unlinked. The body is JSON the schemas do not describe yet.

Status `400`: The account is not linked to this identity.

```json
{
  "error": "string"
}
```

Status `401`: The caller is not signed in.

```json
{
  "error": "string"
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

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
