# bookclerk_plugin_sdk

Bookclerk Python plugin guest SDK.

Provides the workerd guest surface via [`bookclerk_plugin_sdk.workerd`](bookclerk_plugin_sdk.workerd.md).
Authors define a module-level `Default` class extending
[`bookclerk_plugin_sdk.workerd.BookclerkEntrypoint`](bookclerk_plugin_sdk.workerd.md#bookclerkentrypoint) (`event(batch)` /
`job(job)` triggers) plus one class per `plugin.toml` entrypoint
(`class Cli(CliEntrypoint)`, `class Storage(StorageEntrypoint)`, …).
Native guests use the Rust SDK (`serve` / `PluginWorker`). See `docs/plugins/README.md`.

Typical import:

- Workerd: `from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, CliEntrypoint, js`
- Tools: `python -m bookclerk_plugin_sdk check|fmt|types|package|smoke`

Every ABI struct, union, enum, and interface in [`bookclerk_plugin_sdk.abi`](bookclerk_plugin_sdk.abi.md)
and the product constants in `bookclerk_plugin_sdk._abi` are generated
from `crates/bookclerk-plugin-abi/schema/plugin.capnp` — the single ABI
source of truth.

| Name | Kind | Summary |
| --- | --- | --- |
| [`PLUGIN_ERROR_CODES`](#plugin_error_codes) | constant | Stable `PluginError.code` strings. Unknown future codes are forwarded as-is; SDKs surface them as a local `unknown` while keeping the raw wire code. |
| [`PRODUCT_API_VERSION`](#product_api_version) | constant | Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`). |

## Re-exports

- [`Brand`](bookclerk_plugin_sdk.abi.md#brand) from `bookclerk_plugin_sdk.abi`
- [`CliInvokeParams`](bookclerk_plugin_sdk.abi.md#cliinvokeparams) from `bookclerk_plugin_sdk.abi`
- [`CliInvokeResult`](bookclerk_plugin_sdk.abi.md#cliinvokeresult) from `bookclerk_plugin_sdk.abi`
- [`CliSchema`](bookclerk_plugin_sdk.abi.md#clischema) from `bookclerk_plugin_sdk.abi`
- [`D1ExecResult`](bookclerk_plugin_sdk.db_value.md#d1execresult) from `bookclerk_plugin_sdk.db_value`
- [`Entrypoint`](bookclerk_plugin_sdk.abi.md#entrypoint) from `bookclerk_plugin_sdk.abi`
- [`D1Meta`](bookclerk_plugin_sdk.db_value.md#d1meta) from `bookclerk_plugin_sdk.db_value`
- [`D1Result`](bookclerk_plugin_sdk.db_value.md#d1result) from `bookclerk_plugin_sdk.db_value`
- [`DatabaseBinding`](bookclerk_plugin_sdk.db_value.md#databasebinding) from `bookclerk_plugin_sdk.db_value`
- [`DiagnoseResult`](bookclerk_plugin_sdk.abi.md#diagnoseresult) from `bookclerk_plugin_sdk.abi`
- [`FetchTitleParams`](bookclerk_plugin_sdk.abi.md#fetchtitleparams) from `bookclerk_plugin_sdk.abi`
- [`HealthOk`](bookclerk_plugin_sdk.abi.md#healthok) from `bookclerk_plugin_sdk.abi`
- [`Invocation`](bookclerk_plugin_sdk.abi.md#invocation) from `bookclerk_plugin_sdk.abi`
- [`LoginParams`](bookclerk_plugin_sdk.abi.md#loginparams) from `bookclerk_plugin_sdk.abi`
- [`PluginDescribe`](bookclerk_plugin_sdk.abi.md#plugindescribe) from `bookclerk_plugin_sdk.abi`
- [`PluginErrorCode`](bookclerk_plugin_sdk.abi.md#pluginerrorcode) from `bookclerk_plugin_sdk.abi`
- [`RetryToken`](bookclerk_plugin_sdk.db_value.md#retrytoken) from `bookclerk_plugin_sdk.db_value`
- [`ScanParams`](bookclerk_plugin_sdk.abi.md#scanparams) from `bookclerk_plugin_sdk.abi`
- [`canonical_execute_request_hash`](bookclerk_plugin_sdk.db_value.md#canonical_execute_request_hash) from `bookclerk_plugin_sdk.db_value`
- [`create_database_binding`](bookclerk_plugin_sdk.db_value.md#create_database_binding) from `bookclerk_plugin_sdk.db_value`
- [`decode_db_value`](bookclerk_plugin_sdk.db_value.md#decode_db_value) from `bookclerk_plugin_sdk.db_value`
- [`decode_execute_request`](bookclerk_plugin_sdk.db_value.md#decode_execute_request) from `bookclerk_plugin_sdk.db_value`
- [`decode_execute_result_reply`](bookclerk_plugin_sdk.db_value.md#decode_execute_result_reply) from `bookclerk_plugin_sdk.db_value`
- [`encode_db_value`](bookclerk_plugin_sdk.db_value.md#encode_db_value) from `bookclerk_plugin_sdk.db_value`
- [`encode_execute_request`](bookclerk_plugin_sdk.db_value.md#encode_execute_request) from `bookclerk_plugin_sdk.db_value`
- [`encode_execute_result_reply`](bookclerk_plugin_sdk.db_value.md#encode_execute_result_reply) from `bookclerk_plugin_sdk.db_value`
- [`execute_reply_to_d1_results`](bookclerk_plugin_sdk.db_value.md#execute_reply_to_d1_results) from `bookclerk_plugin_sdk.db_value`
- [`parse_db_value`](bookclerk_plugin_sdk.db_value.md#parse_db_value) from `bookclerk_plugin_sdk.db_value`
- [`statement_result_to_d1_result`](bookclerk_plugin_sdk.db_value.md#statement_result_to_d1_result) from `bookclerk_plugin_sdk.db_value`

## Constants

### PLUGIN_ERROR_CODES

```python
PLUGIN_ERROR_CODES: tuple[str, ...] = ('invalid_params', 'unauthorized', 'forbidden', 'not_found', 'unavailable', '...
```

Stable `PluginError.code` strings. Unknown future codes are forwarded
as-is; SDKs surface them as a local `unknown` while keeping the raw wire
code.

Wire strings of `PluginErrorCode` in ordinal order (snake_case `Text` codes).

### PRODUCT_API_VERSION

```python
PRODUCT_API_VERSION: int = 3
```

Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`).
