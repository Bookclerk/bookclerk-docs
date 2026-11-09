# Python plugin SDK

This page shows how to write a Bookclerk plugin in Python with
`bookclerk-plugin-sdk` (`api_version = 3`). A Python plugin is a Python
Worker in a workerd isolate; `bookclerk-workerd` injects
`bookclerk_plugin_sdk.workerd` into the isolate, so you import it rather
than vendor a path.

| Import | What it holds |
| --- | --- |
| `bookclerk_plugin_sdk.workerd` | `BookclerkEntrypoint`, `CliEntrypoint` and the `js` bridge |
| `python -m bookclerk_plugin_sdk` | The author CLI |

## Install

```bash
pip install bookclerk-plugin-sdk          # from PyPI
pip install -e packages/plugin-sdk-python # from this repository
```

Declare Python Workers in `plugin.toml`:

```toml
api_version = 3
runtime = "workerd"
[workerd]
compatibility_flags = ["python_workers", "disable_python_external_sdk"]
main_module = "plugin.py"
```

## Author model

The module-level `Default` class extends `BookclerkEntrypoint` and
implements the triggers `plugin.toml` declares: `event(batch)` for
`[[events.consumers]]`, `job(job)` for `[triggers] jobs`,
`scheduled(controller)` for `[triggers] crons` and
`database_migrations(binding)` for `[[databases]]`. Every named
entrypoint in `entrypoints = [...]` is a module-level class with the
matching name extending its base: `Storefront(StorefrontEntrypoint)`,
`Storage(StorageEntrypoint)`, `RemoteLibrary(RemoteLibraryEntrypoint)`,
`DatabaseAdapter(DatabaseAdapterEntrypoint)`, `Cli(CliEntrypoint)`,
`Oidc(OidcEntrypoint)`, `Metadata(MetadataEntrypoint)` and
`Covers(CoverEntrypoint)`.

```python
from bookclerk_plugin_sdk.workerd import (
    BookclerkEntrypoint,
    CliEntrypoint,
    EventBatch,
    cli_args,
    json_payload,
)


class Cli(CliEntrypoint):
    async def describe(self):
        return {"commands": [{"name": "ping", "about": "Probe", "args": []}]}

    async def invoke(self, params):
        message = cli_args(params).get("message", "hi")
        return {
            "exitCode": 0,
            "stdout": f"pong: {message}\n",
            "stderr": "",
            "payload": json_payload({"pong": message}),
        }


class Default(BookclerkEntrypoint):
    async def event(self, batch: EventBatch):
        for msg in batch.messages:
            if msg.type != "book_acquired":
                msg.reject(f"unexpected {msg.type}")
                continue
            print(f"saw {msg.json().get('titleId', '')} via {self.env.CONFIG}")
            msg.ack()
```

Identity (`apiVersion`, `id`, capabilities) comes from `plugin.toml`; an
optional `describe()` on `Default` only refines `displayName` and similar
fields.

## Triggers

Each `EventMessage` records one outcome, `ack()`, `retry(retry_at=...,
delay_seconds=..., reason=...)`, `reject(reason)`, `dead_letter(reason)`
or `suspend(checkpoint=..., wake_at=...)`, and the first call wins.
Untouched messages ack on return and retry on raise. A `job(job)` handler
returns to complete, raises to reject, or calls `job.suspend(...)` or
`job.retry_later(...)`.

## The `env` bindings

Granted bindings arrive on `self.env` per invocation: `CONFIG`,
`SECRETS`, `EVENTS`, `WORK_FS`, and one entry per `[[databases]]`
binding. A binding's `await limits()` returns the `DbCapabilities` its
session negotiated; a request or result over one of them raises a
`PluginError` whose `code` is `limit_exceeded` and whose `limit` names
the cap. On a host that predates `limits()` the call raises
`unsupported`. `all()` reads through the session's streamed budget when
it advertises one, `prepare(sql).iterate()` yields rows one at a time
from the same read, and `db.chunk(statements)` splits a long list into
batches that each fit one request; see
[Working within database limits](../../database-limits.md). Generate a
typed `Env` protocol from `plugin.toml`:

```bash
python -m bookclerk_plugin_sdk types .   # writes bookclerk_configuration.py
```

## Author tools

```bash
python -m bookclerk_plugin_sdk check .
python -m bookclerk_plugin_sdk types .
python -m bookclerk_plugin_sdk package --out dist .
python -m bookclerk_plugin_sdk smoke .   # download the pinned workerd, then describe + health
```

`smoke` does not need a built `bookclerk-workerd`. It downloads the
pinned Cloudflare `workerd` into `~/.cache/bookclerk/workerd` (override
with `BOOKCLERK_WORKERD_CACHE` or `BOOKCLERK_WORKERD_BIN`).

## Next steps

- [plugin.toml](../../manifest.md): declare the entrypoints and
  triggers the classes above implement.
- [Entrypoints, events and the ABI](../../abi.md): every method and
  event in detail.
- [`examples/plugins-echo-workerd-python`](../../../../examples/plugins-echo-workerd-python/):
  a complete Python plugin.

## Reference

The sections below are generated from the SDK's source: one page per
module, and what the package itself exports.

## Modules

| Module | Summary |
| --- | --- |
| [`bookclerk_plugin_sdk.abi`](abi.md) | GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp. |
| [`bookclerk_plugin_sdk.cli`](cli.md) | CLI entry: `bookclerk-plugin` / `python -m bookclerk_plugin_sdk`. |
| [`bookclerk_plugin_sdk.db_value`](db_value.md) | Universal Cap'n database value domain (`DbValue`) and unpacked codec. |
| [`bookclerk_plugin_sdk.guest_sql`](guest_sql.md) | Guest SQL classification — must match host `guest_statement_kind`. |
| [`bookclerk_plugin_sdk.limits`](limits.md) | Typed `limit_exceeded` errors: the SQL-v1 cap a request or result broke. |
| [`bookclerk_plugin_sdk.path_guard`](path_guard.md) | Path containment helpers for authoring tools and sparse workerd launchers. |
| [`bookclerk_plugin_sdk.plugin_migration`](plugin_migration.md) | Resource bounds for plugin-owned `databaseMigrations` registration. |
| [`bookclerk_plugin_sdk.sparse_workerd`](sparse_workerd/README.md) | Sparse out-of-tree workerd launcher (no Rust `bookclerk-workerd` binary). |
| [`bookclerk_plugin_sdk.sparse_workerd.config`](sparse_workerd/config.md) | Materialize Cap'n Proto workerd config + bridge (mirrors config.rs). |
| [`bookclerk_plugin_sdk.sparse_workerd.ensure`](sparse_workerd/ensure.md) | Download / refresh the pinned Cloudflare `workerd` binary (mirrors ensure.rs). |
| [`bookclerk_plugin_sdk.sparse_workerd.smoke`](sparse_workerd/smoke.md) | Out-of-tree workerd plugin smoke: ensure → materialize → describe + health. |
| [`bookclerk_plugin_sdk.testkit`](testkit.md) | Contract conformance test kit: a plugin database binding like the host's. |
| [`bookclerk_plugin_sdk.tools`](tools.md) | check / fmt / types / package / sync-embed — mirrors Rust/TS author tools. |
| [`bookclerk_plugin_sdk.workerd`](workerd.md) | Workerd author runtime — `BookclerkEntrypoint` and the named entrypoints. |

Bookclerk Python plugin guest SDK.

Provides the workerd guest surface via [`bookclerk_plugin_sdk.workerd`](workerd.md).
Authors define a module-level `Default` class extending
[`bookclerk_plugin_sdk.workerd.BookclerkEntrypoint`](workerd.md#bookclerkentrypoint) (`event(batch)` /
`job(job)` triggers) plus one class per `plugin.toml` entrypoint
(`class Cli(CliEntrypoint)`, `class Storage(StorageEntrypoint)`, …).
Native guests use the Rust SDK (`serve` / `PluginWorker`). See `docs/plugins/README.md`.

Typical import:

- Workerd: `from bookclerk_plugin_sdk.workerd import BookclerkEntrypoint, CliEntrypoint, js`
- Tools: `python -m bookclerk_plugin_sdk check|fmt|types|package|smoke`

Every ABI struct, union, enum, and interface in [`bookclerk_plugin_sdk.abi`](abi.md)
and the product constants in `bookclerk_plugin_sdk._abi` are generated
from `crates/bookclerk-plugin-abi/schema/plugin.capnp` — the single ABI
source of truth.

| Name | Kind | Summary |
| --- | --- | --- |
| [`PLUGIN_ERROR_CODES`](#plugin_error_codes) | constant | Stable `PluginError.code` strings. Unknown future codes are forwarded as-is; SDKs surface them as a local `unknown` while keeping the raw wire code. |
| [`PRODUCT_API_VERSION`](#product_api_version) | constant | Product ABI version (`plugin.toml` `api_version` / `describe().apiVersion`). |

## Re-exports

- [`Brand`](abi.md#brand) from `bookclerk_plugin_sdk.abi`
- [`CliInvokeParams`](abi.md#cliinvokeparams) from `bookclerk_plugin_sdk.abi`
- [`CliInvokeResult`](abi.md#cliinvokeresult) from `bookclerk_plugin_sdk.abi`
- [`CliSchema`](abi.md#clischema) from `bookclerk_plugin_sdk.abi`
- [`D1ExecResult`](db_value.md#d1execresult) from `bookclerk_plugin_sdk.db_value`
- [`Entrypoint`](abi.md#entrypoint) from `bookclerk_plugin_sdk.abi`
- [`D1Meta`](db_value.md#d1meta) from `bookclerk_plugin_sdk.db_value`
- [`D1Result`](db_value.md#d1result) from `bookclerk_plugin_sdk.db_value`
- [`DatabaseBinding`](db_value.md#databasebinding) from `bookclerk_plugin_sdk.db_value`
- [`DiagnoseResult`](abi.md#diagnoseresult) from `bookclerk_plugin_sdk.abi`
- [`ExecuteStreamDecoder`](db_value.md#executestreamdecoder) from `bookclerk_plugin_sdk.db_value`
- [`ExecuteStreamLimits`](db_value.md#executestreamlimits) from `bookclerk_plugin_sdk.db_value`
- [`FetchTitleParams`](abi.md#fetchtitleparams) from `bookclerk_plugin_sdk.abi`
- [`HealthOk`](abi.md#healthok) from `bookclerk_plugin_sdk.abi`
- [`Invocation`](abi.md#invocation) from `bookclerk_plugin_sdk.abi`
- [`LoginParams`](abi.md#loginparams) from `bookclerk_plugin_sdk.abi`
- [`PluginDescribe`](abi.md#plugindescribe) from `bookclerk_plugin_sdk.abi`
- [`PluginErrorCode`](abi.md#pluginerrorcode) from `bookclerk_plugin_sdk.abi`
- [`RetryToken`](db_value.md#retrytoken) from `bookclerk_plugin_sdk.db_value`
- [`ScanParams`](abi.md#scanparams) from `bookclerk_plugin_sdk.abi`
- [`canonical_execute_request_hash`](db_value.md#canonical_execute_request_hash) from `bookclerk_plugin_sdk.db_value`
- [`chunk_indexes`](db_value.md#chunk_indexes) from `bookclerk_plugin_sdk.db_value`
- [`create_database_binding`](db_value.md#create_database_binding) from `bookclerk_plugin_sdk.db_value`
- [`decode_db_value`](db_value.md#decode_db_value) from `bookclerk_plugin_sdk.db_value`
- [`decode_execute_request`](db_value.md#decode_execute_request) from `bookclerk_plugin_sdk.db_value`
- [`decode_db_capabilities_reply`](db_value.md#decode_db_capabilities_reply) from `bookclerk_plugin_sdk.db_value`
- [`decode_execute_result_reply`](db_value.md#decode_execute_result_reply) from `bookclerk_plugin_sdk.db_value`
- [`decode_execute_stream`](db_value.md#decode_execute_stream) from `bookclerk_plugin_sdk.db_value`
- [`encode_db_capabilities_reply`](db_value.md#encode_db_capabilities_reply) from `bookclerk_plugin_sdk.db_value`
- [`LimitExceeded`](limits.md#limitexceeded) from `bookclerk_plugin_sdk.limits`
- [`limit_exceeded_message`](limits.md#limit_exceeded_message) from `bookclerk_plugin_sdk.limits`
- [`parse_limit_exceeded`](limits.md#parse_limit_exceeded) from `bookclerk_plugin_sdk.limits`
- [`encode_db_value`](db_value.md#encode_db_value) from `bookclerk_plugin_sdk.db_value`
- [`encode_execute_request`](db_value.md#encode_execute_request) from `bookclerk_plugin_sdk.db_value`
- [`encode_execute_result_reply`](db_value.md#encode_execute_result_reply) from `bookclerk_plugin_sdk.db_value`
- [`execute_reply_to_d1_results`](db_value.md#execute_reply_to_d1_results) from `bookclerk_plugin_sdk.db_value`
- [`execute_stream_limits`](db_value.md#execute_stream_limits) from `bookclerk_plugin_sdk.db_value`
- [`raw_stream_ceiling`](db_value.md#raw_stream_ceiling) from `bookclerk_plugin_sdk.db_value`
- [`streams_results`](db_value.md#streams_results) from `bookclerk_plugin_sdk.db_value`
- [`parse_db_value`](db_value.md#parse_db_value) from `bookclerk_plugin_sdk.db_value`
- [`statement_result_to_d1_result`](db_value.md#statement_result_to_d1_result) from `bookclerk_plugin_sdk.db_value`

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
