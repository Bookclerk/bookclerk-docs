# Accounts

Store accounts a library user connects (the Accounts page).

Back to the [web API reference](README.md).

## `POST /api/portal/redeem`

Redeems a claim ticket, optionally setting a first password, and sets the session cookie.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`RedeemBody`](schemas.md#redeembody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in; the response sets the portal session cookie. | any JSON (`application/json`) |
| 400 | The ticket is invalid, used or expired. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/portal/login/integration`

Authenticates against an enabled integration and mints a portal session.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`IntegrationLoginBody`](schemas.md#integrationloginbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Signed in through the integration. | any JSON (`application/json`) |
| 400 | The request is invalid, or the integration refused the login. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The account was connected. | any JSON (`application/json`) |
| 400 | The login failed. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

## `POST /api/portal/libro/login`

**Deprecated.**

Legacy `POST /libro/login` alias that forwards to the Libro password-login handler.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `portalSession`

### Request body

`application/json`: [`PasswordLoginBody`](schemas.md#passwordloginbody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The account was connected. | any JSON (`application/json`) |
| 401 | The caller is not signed in. | [`PortalErrorBody`](schemas.md#portalerrorbody) (`application/json`) |
| 403 | A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

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

## `POST /api/portal/sources/{source}/connections/{account_id}/revoke`

Unlinks a store account from this identity and deletes secrets when no other identity remains.

The account is addressed by `(source, account_id)`: the same store id may
exist under two plugins. `source` is the alias or plugin key of the link.

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
