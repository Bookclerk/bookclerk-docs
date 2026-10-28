# Spec

This document.

Back to the [web API reference](README.md).

## `GET /api/openapi.json`

Serves the OpenAPI document.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The OpenAPI 3.1 document for this daemon's version. | any JSON (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/openapi.json'
```

## `GET /api/openapi-3.0.json`

Serves the OpenAPI document restated as OpenAPI 3.0.3.

The same API as `GET /api/openapi.json`, restated in OpenAPI 3.0.3 form for tools that only read 3.0, such as Cloudflare API Shield's schema validation: nullable types use `nullable: true`, and a nullable reference is a copy of the schema it names.

**Access:** No auth layer. Routes that need a caller say so in `security`; the handler checks it.

**Credentials:** none

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The OpenAPI 3.0.3 document for this daemon's version. | any JSON (`application/json`) |
| 500 | The document could not be restated; this is a bug. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

### Example request

```console
$ curl 'http://127.0.0.1:8787/api/openapi-3.0.json'
```

### Example response

Status `200`: The OpenAPI 3.0.3 document for this daemon's version. The body is JSON the schemas do not describe yet.

Status `500`: The document could not be restated; this is a bug.

```json
{
  "error": "string",
  "message": "string",
  "status": 0
}
```

Status `504`: The request ran past the API's time budget. The body is `text/plain`.
