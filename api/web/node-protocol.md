# Node protocol

Node-to-node enrollment, socket, artifacts and uploads; each route checks its own credential.

Back to the [web API reference](README.md).

## `POST /api/node/v1/enroll`

`POST /api/node/v1/enroll`: redeem an enrollment link. Answers
`{cluster_id, node_id}`; 403 `transport_insecure` when the connection
fails the transport policy, 403 `enrollment_refused` for any link that
does not redeem.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`EnrollBody`](schemas.md#enrollbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `link_id` | string | yes | Link id from the token. |
| `secret` | string | yes | Link secret from the token. |
| `sign_pk` | string | yes | Ed25519 public signing key. |
| `box_pk` | string | yes | X25519 public box key. |
| `software_version` | string | no | Host's Bookclerk version. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The host is enrolled; its node credential. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The link is unknown, used or expired. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/enroll' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"link_id": "string", "secret": "string", "sign_pk": "string", "box_pk": "string"}'
```

## `GET /api/node/v1/info`

`GET /api/node/v1/info`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | What this library says about itself to a joining install. | any JSON (`application/json`) |
| 403 | The transport is not allowed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/info'
```

## `POST /api/node/v1/enroll/oauth`

`POST /api/node/v1/enroll/oauth`: redeem an approved branch sign-in.
Answers `{cluster_id, node_id}`; 403 `transport_insecure` for a refused
connection, 403 `enrollment_refused` for a code that does not redeem
(unknown, used, expired, another client's, a wrong verifier or
redirect URI, enrollment off, or the user no longer active).

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Request body

`application/json`: [`OAuthEnrollRequest`](schemas.md#oauthenrollrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string | yes | The authorization code from the callback. |
| `code_verifier` | string | yes | The PKCE verifier. |
| `redirect_uri` | string | yes | The redirect URI the authorization was requested with. |
| `sign_pk` | string | yes | Ed25519 public signing key. |
| `box_pk` | string | yes | X25519 public box key. |
| `branch_name` | string | no | The name this branch joins as (the node's label). |
| `software_version` | string | no | The joining install's Bookclerk version. |
| `connection` | [`ConnectionType`](schemas.md#connectiontype) | no | The connection type the code was approved for. It must match the one bound to the code at consent; left out for a branch. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The branch is enrolled; its node credential. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The code or verifier is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The transport is not allowed. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | The enrollment could not be recorded; try again (`enrollment_unavailable`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/enroll/oauth' \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"code": "string", "code_verifier": "string", "redirect_uri": "string", "sign_pk": "string", "box_pk": "string"}'
```

## `GET /api/node/v1/connect`

`GET /api/node/v1/connect`: upgrade to the host-node socket after the
transport check. 403 `transport_insecure` for a refused connection,
400 `subprotocol_required` without [`SUBPROTOCOL`].

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 101 | Switching to the host-node WebSocket. The node's signature is checked in the first frames; a wrong one closes the socket. |  |
| 400 | The request does not offer the node subprotocol (`subprotocol_required`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The transport is not allowed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | Too many handshakes are in flight (`handshakes_busy`); retry after `Retry-After` seconds. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/connect'
```

## `GET /api/node/v1/artifacts/{sha256}`

`GET /api/node/v1/artifacts/{sha256}`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `nodeSignature`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `sha256` | path | string | yes | Archive digest, lowercase hex. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The plugin archive. | [`BinaryBody`](schemas.md#binarybody) (`application/octet-stream`) |
| 401 | The node signature is missing or wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such archive, or the node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/artifacts/<sha256>' \
    -H 'Authorization: Bookclerk-Node <node_id> <issued_at> <signature>'
```

## `PUT /api/node/v1/uploads/{upload_id}/files/{index}`

`PUT /api/node/v1/uploads/{upload_id}/files/{index}`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `uploadToken`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `upload_id` | path | string | yes | Upload id. |
| `index` | path | integer (int32) | yes | File index in the upload. |

### Request body

`application/octet-stream`: [`BinaryBody`](schemas.md#binarybody)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The file was stored (`received`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 401 | The upload token is missing or wrong (`unauthorized`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 403 | The connection's transport is refused (`transport_insecure`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such upload, or the node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The upload lost its fence to a newer one (`lost`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 421 | This node is not the upload's receiving node (`misdirected`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 422 | The file was refused (`refused`): not in the upload (`unknown_file`), a size or content mismatch (`size_mismatch`, `content_mismatch`), the destinations changed (`destinations_changed`), or a granted storage key now belongs to another title or writer (`key_taken`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 503 | Storage or the upload record is unavailable, or the file lock timed out; retry (`unavailable`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/files/<index>' \
    -X PUT \
    -H 'Authorization: Bookclerk-Upload <token>' \
    -H 'Content-Type: application/octet-stream' \
    --data-binary @file
```

### Example response

Status `200`:

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
}
```

## `POST /api/node/v1/uploads/{upload_id}/commit`

`POST /api/node/v1/uploads/{upload_id}/commit`.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** `uploadToken`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `upload_id` | path | string | yes | Upload id. |

### Request body

`application/json`: [`UploadCommitRequest`](schemas.md#uploadcommitrequest)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `finish` | boolean | no | Complete the job with this commit (the job's closing upload). |
| `detail` | string or null | no | Outcome text recorded as the job's progress when `finish` is set. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The upload was committed to the destinations (`committed`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 401 | The upload token is missing or wrong (`unauthorized`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 403 | The connection's transport is refused (`transport_insecure`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such upload, or the node API is not mounted. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The upload lost its fence to a newer one (`lost`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 421 | This node is not the upload's receiving node (`misdirected`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 422 | The commit was refused (`refused`): an invalid body, missing files, or the destinations changed. | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 503 | Storage or the upload record is unavailable; retry (`unavailable`). | [`UploadAnswer`](schemas.md#uploadanswer) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/node/v1/uploads/<upload_id>/commit' \
    -X POST \
    -H 'Authorization: Bookclerk-Upload <token>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

### Example response

Status `200`:

```json
{
  "status": "string",
  "reason": "string",
  "job_completed": true
}
```
