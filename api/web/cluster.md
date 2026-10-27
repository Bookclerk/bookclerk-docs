# Cluster

Nodes, clerks, branches, joining a library and host enrollment.

Back to the [web API reference](README.md).

## `PUT /api/config/nodes/{node_id}/labels`

`PUT /api/config/nodes/{node_id}/labels`: replace a node's placement
labels. The scheduler re-plans on the wakeup this publishes.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `node_id` | path | string | yes | Node id. |

### Request body

`application/json`: [`PutNodeLabels`](schemas.md#putnodelabels)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `labels` | array of string | yes | The node's complete operator label set. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The node's labels after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such node, or not one the caller administers. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/labels' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"labels": ["string"]}'
```

## `PUT /api/config/nodes/{node_id}/owner`

`PUT /api/config/nodes/{node_id}/owner`: assign or clear a node's owner.
The owner may then edit that node's runtime settings without operator
authority.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `node_id` | path | string | yes | Node id. |

### Request body

`application/json`: [`PutNodeOwner`](schemas.md#putnodeowner)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) or null (int64) | no | User who owns the node, or `null` to clear. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The node's owner after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such node, or not one the caller administers. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/owner' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `GET /api/config/cluster-consents`

`GET /api/config/cluster-consents`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Pending and granted cluster consents. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/config/clusters/{cluster_id}/scan`

`POST /api/config/clusters/{cluster_id}/scan`: asks the owner cluster
to queue a scan for this node's owner (H-5). The owner's planner decides
what that user may scan; the answer lists the queued job ids.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `cluster_id` | path | string | yes | Owner cluster id. |

### Request body

Optional.

`application/json`: [`ClusterScanBody`](schemas.md#clusterscanbody) or null

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `account` | string or null | no | Scan only this store account of the owner cluster. |
| `plugin_instance_id` | string or null | no | Pin every planned job to this owner plugin instance. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The owner cluster accepted the scan. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such cluster. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The owner cluster could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/scan' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `POST /api/config/clusters/{cluster_id}/jobs`

`POST /api/config/clusters/{cluster_id}/jobs`: the owner's view of jobs
this node may see (those it would be trusted to claim).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `cluster_id` | path | string | yes | Owner cluster id. |

### Request body

`application/json`: [`ClusterJobsBody`](schemas.md#clusterjobsbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `job_ids` | array of string | yes | Job ids on the owner cluster. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The owner's jobs for these accounts. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such cluster. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The owner cluster could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clusters/<cluster_id>/jobs' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"job_ids": ["string"]}'
```

## `POST /api/config/cluster-consents/approve`

`POST /api/config/cluster-consents/approve`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`ApproveBody`](schemas.md#approvebody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | Owner cluster id. |
| `plugin_key` | string | yes | Canonical plugin key. |
| `request_digest` | string | yes | The `request_digest` of the request the operator reviewed, as the listing shows it (refused when the recorded request is another one). |
| `deny_domains` | array of string | no | Domains to take out of the request. No duplicates. |
| `deny_bindings` | array of string | no | Bindings to take out of the request. No duplicates. |
| `deny_network` | boolean | no | Deny all network access. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The consent was granted. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such consent. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents/approve' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "plugin_key": "string", "request_digest": "string"}'
```

## `POST /api/config/cluster-consents/revoke`

`POST /api/config/cluster-consents/revoke`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`RevokeBody`](schemas.md#revokebody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | Owner cluster id. |
| `plugin_key` | string | yes | Canonical plugin key. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The consent was revoked. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such consent. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/cluster-consents/revoke' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "plugin_key": "string"}'
```

## `GET /api/config/join`

`GET /api/config/join`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Whether this install has joined a library, and its state. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/config/join/inspect`

`POST /api/config/join/inspect`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`InspectBody`](schemas.md#inspectbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `address` | string | yes | The library's address (`https://owner.example`, `owner.lan:8443`). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | What the library says about itself. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The library could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/inspect' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"address": "string"}'
```

## `POST /api/config/join/start`

`POST /api/config/join/start`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`StartBody`](schemas.md#startbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `address` | string | yes | The library's address. |
| `fingerprint` | string or null | no | The fingerprint the operator confirmed (self-signed libraries). |
| `branch_name` | string or null | no | The name to join as (default: this machine's name). |
| `clerk` | boolean | no | Offer to join as a clerk (an owner node that shares the library's database): only an empty install may, and its data is replaced. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The library's sign-in URL to open. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | A join is already in progress or done. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The library could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/start' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"address": "string"}'
```

## `POST /api/config/join/callback`

`POST /api/config/join/callback`: redeems the library's code with the
sign-in's sealed verifier and keys. The pending sign-in is kept when the
redeem fails for a reason a retry can fix (answered as 502
`join_retry`; the page offers Try again with the same code), and ended
on success, refusal, or anything else.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CallbackBody`](schemas.md#callbackbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `code` | string | no | The code from the library. |
| `state` | string | no | The `state` it came back with. |
| `error` | string or null | no | The library's `error` (`access_denied`), when it sent one. |
| `connection` | string or null | no | The connection type the library's user chose (`clerk`; a branch when left out). The code is bound to it, so a changed value only fails the redeem. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The join completed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route, or the library refused the sign-in (`denied`). A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 410 | The sign-in is unknown, already used, or older than ten minutes; start again (`join_expired`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The library could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/callback' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `POST /api/config/join/invite`

`POST /api/config/join/invite`: join with a pasted branch invite. The
link is redeemed once and dropped; it is never logged or echoed. A
library slower than [`REDEEM_TIMEOUT`] is answered 504 `join_pending`
while the join carries on and starts its client when it lands.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`InviteBody`](schemas.md#invitebody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `link` | string | yes | The whole branch invite (`https://<library>/enroll#bcn1...`). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The join completed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The library could not be reached. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The library is slow to answer; the join goes on in the background (`join_pending`). Poll the join state instead of retrying. | [`ApiError`](schemas.md#apierror) (`application/json`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/invite' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"link": "string"}'
```

## `POST /api/config/join/leave`

`POST /api/config/join/leave`: Leave library. The leave runs on its own
task, so a request cut short (the client gone, or the API deadline)
still records it once the library has been told.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`LeaveBody`](schemas.md#leavebody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | The library (owner cluster id) to leave. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Leaving started. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | This install is not a branch of that library (`not_found`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | This install has not joined a library. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/leave' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string"}'
```

## `GET /api/config/nodes`

`GET /api/config/nodes`: every node for an operator; the caller's own
nodes for a node owner.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The nodes the caller administers. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `DELETE /api/config/nodes/{node_id}`

`DELETE /api/config/nodes/{node_id}`: remove an owner node (a clerk, or
any node sharing this database) that is no longer running from the
library's node list. The operator or an Owner may remove any; another
user only one they own. A running node, this node, or a branch is
refused (a branch is revoked instead).

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `node_id` | path | string | yes | Node id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The owner node was removed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such node, or not one the caller administers. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

## `GET /api/config/clerks`

`GET /api/config/clerks`: the owner nodes (this one and its clerks) for
Settings → Clerks, shaped like `GET /api/config/nodes`. The operator or
an Owner sees every one, as `DELETE /api/config/nodes/{node_id}` lets
them remove any; another user only the ones they own, which the nodes
listing already shows them. Branches are not listed.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | This node and its clerks. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/clerks' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `PUT /api/config/nodes/{node_id}/runtime`

`PUT /api/config/nodes/{node_id}/runtime`: edit a node's label, budgets,
or drain switch. Applies at that node's next heartbeat. A node owner may
edit only the nodes it owns.

**Access:** The operator, a signed-in Owner, Administrator or Host, or a user who owns at least one node.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `node_id` | path | string | yes | Node id. |

### Request body

`application/json`: [`PutNodeRuntime`](schemas.md#putnoderuntime)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `label` | string or null | no | Operator label. |
| `cpu_budget_percent` | integer (int32) or null (int32) | no | Share of detected CPU the scheduler may commit (1..=100). At least 0. |
| `memory_budget_percent` | integer (int32) or null (int32) | no | Share of detected memory the scheduler may commit (1..=100). At least 0. |
| `schedulable` | boolean or null | no | False drains the node. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The node's runtime settings after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such node, or not one the caller administers. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/nodes/<node_id>/runtime' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `GET /api/config/node-invites`

`GET /api/config/node-invites`: every link for the operator or an
Owner; the caller's own links for anyone else, and with `?mine=true`.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `mine` | query | boolean | no | Only the links for the calling user's own branches (an Owner's "Add a branch" dialog), even when the caller may see everyone's. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Enrollment links the caller may see. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-invites' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/config/node-invites`

`POST /api/config/node-invites`: mint an enrollment link. Any user may
mint one for themselves while host enrollment is on; the operator or an
Owner may mint for any active user.

### Arguments

* `caller` - Who is asking; decides whose links they may mint.
* `body` - Owner, lifetime, and label of the new link.

### Returns

`{link, url}`; the secret appears once, in `url`.

### Errors

Returns 503 `node_api_pin_unavailable` in `self_signed` mode while the
certificate's SPKI pin cannot be read (nothing is minted), and an error
response when the library refuses the mint.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PostNodeInvite`](schemas.md#postnodeinvite)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user_id` | integer (int64) or null (int64) | no | User who will own the node. Defaults to the caller; the operator must name one. |
| `ttl_secs` | integer (int64) or null (int64) | no | Link lifetime in seconds (default 24 h, 5 min to 7 days). At least 0. |
| `label` | string | no | Label shown in listings. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new link (shown once). | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | Host enrollment is off. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 503 | The node API's certificate pin is not available yet (`node_api_pin_unavailable`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-invites' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `DELETE /api/config/node-invites/{id}`

`DELETE /api/config/node-invites/{id}`: revoke an unredeemed link.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Link id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The link was revoked. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such link. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-invites/<id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

## `GET /api/config/branches`

`GET /api/config/branches`: this library's join settings: whether new
branches may join, the transport mode, and the SPKI fingerprint a
joining operator confirms in `self_signed` mode.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The library's join settings and branches. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branches' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `PUT /api/config/branches`

`PUT /api/config/branches`: turn "Let new branches join" on or off.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PutBranches`](schemas.md#putbranches)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enrollment` | boolean | yes | "Let new branches join" (`core.host_enrollment.enabled`). |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The settings after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branches' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enrollment": true}'
```

## `POST /api/config/branches/titles/remove`

`POST /api/config/branches/titles/remove`: removes the entries of
titles from a branch that stopped sharing or left. Delivered files stay.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`RemoveBranchTitles`](schemas.md#removebranchtitles)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `node_id` | string or null | no | Only this branch's entries; every branch's when absent. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The branch's titles were removed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/branches/titles/remove' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{}'
```

## `GET /api/config/join/sharing`

`GET /api/config/join/sharing`: Settings → Library for the operator or
an Owner: every library this install is a branch of, with its two
sharing settings (`{clusters: [...]}`).

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The branch's sharing settings. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/sharing' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `POST /api/config/join/sharing`

`POST /api/config/join/sharing`: Settings → Library's two sharing
settings for one library this install is a branch of. The operator or
an Owner (verified from the stored role); anyone else gets 403.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`SharingBody`](schemas.md#sharingbody)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cluster_id` | string | yes | The library (owner cluster id). |
| `share_books` | boolean | yes | "Share this branch's books with *library*". |
| `include_branches` | boolean | yes | "Include books from my own branches". |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The sharing settings after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | This install is not a branch of that library (`not_found`). | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/join/sharing' \
    -X POST \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"cluster_id": "string", "share_books": true, "include_branches": true}'
```

## `DELETE /api/config/node-credentials/{node_id}`

`DELETE /api/config/node-credentials/{node_id}`: revoke an enrolled
host's credential so it can no longer connect.

### Arguments

* `caller` - Who is asking; the operator or an Owner may revoke any host,
  any other active user only the hosts they own.
* `node_id` - The host node to revoke.

### Returns

`{credential}` with `revoked_at` set; revoking twice keeps the first
time.

### Errors

Returns 404 when the node is unknown or not the caller's, 400 when the
node has no host credential, 403 for a disabled user, and an error
response when the write fails.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `node_id` | path | string | yes | Node id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The host's credential was revoked. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such host. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/node-credentials/<node_id>' \
    -X DELETE \
    -H 'Authorization: Bearer <operatorToken>'
```

## `GET /api/config/host-enrollment`

`GET /api/config/host-enrollment`: whether host nodes may enroll.

### Returns

`{enabled, revision}`; `revision` is `null` while the default (enabled)
applies.

### Errors

Returns an error response when the switch cannot be read.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Whether host nodes may enroll. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/host-enrollment' \
    -H 'Authorization: Bearer <operatorToken>'
```

## `PUT /api/config/host-enrollment`

`PUT /api/config/host-enrollment`: turn host enrollment on or off. The
operator may always; a user only while the stored role is an active
Owner.

### Arguments

* `caller` - Who is asking; a user's role is read from the database.
* `body` - The new switch value.

### Returns

`{enabled, revision}` as stored.

### Errors

Returns 403 when the caller is neither the operator nor an active Owner,
and an error response when the switch cannot be written.

**Access:** The operator, or any signed-in user.

**Credentials:** `operatorToken` or `operatorSession` or `portalSession`

### Request body

`application/json`: [`PutHostEnrollment`](schemas.md#puthostenrollment)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `enabled` | boolean | yes | Whether host nodes may enroll into this cluster. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The switch after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller may not do this. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/config/host-enrollment' \
    -X PUT \
    -H 'Authorization: Bearer <operatorToken>' \
    -H 'Content-Type: application/json' \
    -d '{"enabled": true}'
```
