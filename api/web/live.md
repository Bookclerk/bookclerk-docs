# Live

The web interface's live-updates socket.

Back to the [web API reference](README.md).

## `GET /api/live`

`GET /api/live`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 101 | Switching to the live-updates WebSocket. |  |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The Origin is not allowed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/live' \
    -H 'Authorization: Bearer <operatorToken>'
```

### Example response

Status `101`: Switching to the live-updates WebSocket. No body.

Status `401`: The caller is not signed in.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `403`: The Origin is not allowed.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
