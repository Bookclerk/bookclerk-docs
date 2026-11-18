# Library copies

Downloads of an edition's files the library already holds, by a branch or clerk it granted the copy to; the grant's token is the credential.

Back to the [web API reference](README.md).

## `GET /api/copies/v1/{copy_id}/files/{index}`

Streams one file of a granted copy.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `copyToken`

### Parameters

- `copy_id` (path) *string*, **required** — Copy id from `copy.granted`, on the node socket or the clerk socket.
- `index` (path) *integer (int32)*, **required** — The file's index in the grant's `files`.
- `Range` (header) *string or null* — One byte range (`bytes=A-B`, `bytes=A-` or `bytes=-N`); anything else reads the whole file.

### Responses

- `200` — The whole file. Body: [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `206` — The requested byte range, with `Content-Range`. Body: [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`)
- `401` — The copy token is missing, wrong or expired, or the grant no longer holds: the branch is no longer trusted or the library card is signed out, the person may no longer take the edition, or the library's files changed since the grant (`unauthorized`). Ask for a new grant; a clerk whose library card is signed out signs in again first. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `403` — The connection failed the transport policy (`transport_insecure`, with the check in `reason`). Body: [`TransportRefused`](schemas.md#transportrefused) (`application/json`)
  - `error` *string*, **required** — Always `transport_insecure`.
  - `reason` *string*, **required** — The transport check that refused the connection (`plain_http`, `unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`, `proxy_proto_ambiguous`, `proxy_proto_not_https`, `public_origin_not_https`, `listener_mode_mismatch`, `node_api_misconfigured`).
- `404` — The index names no file of the copy (`unknown_file`), or the node API is not mounted on this listener (`node_api_unmounted`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `416` — The range starts at or past the end of the file (`range_not_satisfiable`); `Content-Range` states the size. Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `421` — Another node granted this copy (`misdirected`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `503` — Storage or the grant record is unavailable; retry (`unavailable`). Body: [`ErrorMessage`](schemas.md#errormessage) (`application/json`)
  - `error` *string*, **required** — What went wrong, as the handler describes it.
- `504` — The request ran past the API's time budget. Body: string (`text/plain`)

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/copies/v1/<copy_id>/files/<index>' \
    -H 'Authorization: Bookclerk-Copy <token>'
```

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:8787/api/copies/v1/<copy_id>/files/<index>' `
    -Headers @{ Authorization = 'Bookclerk-Copy <token>' }
```

```python
import requests

response = requests.get(
    "http://127.0.0.1:8787/api/copies/v1/<copy_id>/files/<index>",
    headers={"Authorization": "Bookclerk-Copy <token>"},
)
print(response.status_code, response.text)
```

```javascript
const response = await fetch("http://127.0.0.1:8787/api/copies/v1/<copy_id>/files/<index>", {
  headers: { Authorization: "Bookclerk-Copy <token>" },
});
console.log(response.status, await response.text());
```

```rust
let client = reqwest::Client::new();
let response = client
    .get("http://127.0.0.1:8787/api/copies/v1/<copy_id>/files/<index>")
    .header("Authorization", "Bookclerk-Copy <token>")
    .send()
    .await?;
println!("{} {}", response.status(), response.text().await?);
```

### Example response

Status `200`: The whole file. The body is binary (`application/octet-stream`).

Schema:

A `application/octet-stream` body: binary bytes.

Status `206`: The requested byte range, with `Content-Range`. The body is binary (`application/octet-stream`).

Schema:

A `application/octet-stream` body: binary bytes.

Status `401`: The copy token is missing, wrong or expired, or the grant no longer holds: the branch is no longer trusted or the library card is signed out, the person may no longer take the edition, or the library's files changed since the grant (`unauthorized`). Ask for a new grant; a clerk whose library card is signed out signs in again first.

```json
{
  "error": "unauthorized"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `403`: The connection failed the transport policy (`transport_insecure`, with the check in `reason`).

```json
{
  "error": "transport_insecure",
  "reason": "plain_http"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON body of a `403` from the node API's transport policy: the\nconnection is plain HTTP off loopback, or arrived through a proxy the\npolicy does not trust, and `reason` names the check that refused it.",
  "required": [
    "error",
    "reason"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "Always `transport_insecure`.",
      "example": "transport_insecure"
    },
    "reason": {
      "type": "string",
      "description": "The transport check that refused the connection (`plain_http`,\n`unknown_peer`, `untrusted_forwarding`, `proxy_proto_missing`,\n`proxy_proto_ambiguous`, `proxy_proto_not_https`,\n`public_origin_not_https`, `listener_mode_mismatch`,\n`node_api_misconfigured`).",
      "example": "plain_http"
    }
  }
}
```

Status `404`: The index names no file of the copy (`unknown_file`), or the node API is not mounted on this listener (`node_api_unmounted`).

```json
{
  "error": "unknown_file"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `416`: The range starts at or past the end of the file (`range_not_satisfiable`); `Content-Range` states the size.

```json
{
  "error": "range_not_satisfiable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `421`: Another node granted this copy (`misdirected`).

```json
{
  "error": "misdirected"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `503`: Storage or the grant record is unavailable; retry (`unavailable`).

```json
{
  "error": "unavailable"
}
```

Schema:

```json
{
  "type": "object",
  "description": "JSON error object with a single `error` message, written by handlers\nthat report a library or plugin error in their own words.",
  "required": [
    "error"
  ],
  "properties": {
    "error": {
      "type": "string",
      "description": "What went wrong, as the handler describes it.",
      "example": "plugin instance 3f9c2b7e-1d4a-4c8e-9b2f-6a7d8e9f0a1b not found"
    }
  }
}
```

Status `504`: The request ran past the API's time budget.

```text
request timed out
```

Schema:

A `text/plain` body: a string.
