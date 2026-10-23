# Plugins

Plugin packages, instances, consent and deployment.

Back to the [web API reference](README.md).

## `GET /api/config/plugin-instances`

`GET /api/config/plugin-instances`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Every plugin instance with its deployments. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/config/plugin-instances`

`POST /api/config/plugin-instances`: a second (non-default) instance of
a plugin, with an empty document so schema defaults apply. With
`node_id`, the instance is local to that node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`CreatePluginInstance`](schemas.md#createplugininstance)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The new instance. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance id is taken. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/config/plugin-instances/{id}`

`DELETE /api/config/plugin-instances/{id}`: removes an instance that has
no deployment left.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance was removed. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance is a default or still deployed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/config/plugin-instances/{id}/config`

`GET /api/config/plugin-instances/{id}/config`.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance's settings document (secrets as presence flags). | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/config/plugin-instances/{id}/config`

`PUT /api/config/plugin-instances/{id}/config`.

Operator principal only. The route is mounted on the operator router.
The document is validated against the plugin's pinned schema, so this
node does not need the guest installed.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Request body

`application/json`: [`PutPluginInstanceConfig`](schemas.md#putplugininstanceconfig)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The document after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/config/plugin-instances/{id}/deployments/{node_id}`

`PUT /api/config/plugin-instances/{id}/deployments/{node_id}`: the
Deployed switch. Creates the row when the node has none.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |
| `node_id` | path | string | yes | Node id. |

### Request body

`application/json`: [`PutPluginDeployment`](schemas.md#putplugindeployment)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The deployment after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The instance is placed by the scheduler. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/config/plugin-instances/{id}/placement`

`PUT /api/config/plugin-instances/{id}/placement`: hand the instance's
deployments to the scheduler with a replica count and constraints.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Request body

`application/json`: [`PutPluginPlacement`](schemas.md#putpluginplacement)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The scheduler now places the instance. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/config/plugin-instances/{id}/placement`

`DELETE /api/config/plugin-instances/{id}/placement`: return the
instance to hand-managed deployments; its deployments stay as they
are.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin instance id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The instance is back to manual deployment. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such instance. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/config/plugins`

`GET /api/config/plugins`: every installed plugin.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | Every installed plugin. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/config/plugins`

`POST /api/config/plugins`: install (and maybe activate) a version.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Request body

`application/json`: [`PostPluginInstall`](schemas.md#postplugininstall)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The installed version. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | A local package is missing its `package.json` or `archive.tar.gz`. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 409 | The version is already installed. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 502 | The package catalog could not be reached or answered badly. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/config/plugins/{key}`

`GET /api/config/plugins/{key}`: one installed plugin with its pinned
settings schema.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The plugin with its pinned versions. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `DELETE /api/config/plugins/{key}`

`DELETE /api/config/plugins/{key}`: uninstall the plugin (active version
cleared, placements dropped, every deployment `absent`).

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The plugin was uninstalled. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `PUT /api/config/plugins/{key}/active`

`PUT /api/config/plugins/{key}/active`: activate, update, or roll back.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |

### Request body

`application/json`: [`PutActiveVersion`](schemas.md#putactiveversion)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The active version after the change. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/config/plugins/{key}/versions/{version}/approve`

`POST /api/config/plugins/{key}/versions/{version}/approve`: record the
version's requested consent as approved for every node.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `key` | path | string | yes | Plugin key. |
| `version` | path | string | yes | Semver version. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The version's grant was approved. | any JSON (`application/json`) |
| 400 | The body is invalid. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 500 | The configuration could not be read or written. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/plugins/{id}/consent`

Loads the consent dialog for one plugin, including coverage against the stored grant.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin id. |
| `plugin_key` | query | string | no | Canonical PluginKey; when set, wins over the path alias. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | What the plugin asks for and what is granted. | [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The grant could not be read. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `POST /api/plugins/{id}/consent`

Persists an approved plugin grant after validating operator overrides.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `id` | path | string | yes | Plugin id. |
| `plugin_key` | query | string | no | Canonical PluginKey; when set, wins over the path alias. |

### Request body

`application/json`: [`PluginConsentApproveRequest`](schemas.md#pluginconsentapproverequest)

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The grant was stored. | [`PluginConsentResponse`](schemas.md#pluginconsentresponse) (`application/json`) |
| 400 | An override is not allowed. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. A write sent with a session cookie but without a matching `Origin` or `Referer` is refused. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | No such plugin. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 500 | The grant could not be stored. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |

## `GET /api/plugins/{kind}/{id}/logo`

Serve an embedded `plugin.toml` `logo` file from under the plugin install root.

**Access:** The operator (token or session) only.

**Credentials:** `operatorToken` or `operatorSession`

### Parameters

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `kind` | path | string | yes | Plugin kind (`source`, `destination`, `integration`, …). |
| `id` | path | string | yes | Plugin id. |

### Responses

| Status | Description | Body |
| --- | --- | --- |
| 200 | The logo (PNG, or SVG served with a restrictive CSP). | [`BinaryBody`](schemas.md#binarybody) (`image/png`); [`BinaryBody`](schemas.md#binarybody) (`image/svg+xml`) |
| 401 | The caller is not signed in, or the credential is wrong. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 403 | The caller's role does not allow this route. | [`ApiError`](schemas.md#apierror) (`application/json`) |
| 404 | The plugin has no logo. | [`ErrorJson`](schemas.md#errorjson) (`application/json`) |
| 504 | The request ran past the API's time budget. | string (`text/plain`) |
