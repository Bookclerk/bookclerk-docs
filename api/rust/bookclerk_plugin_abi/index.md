# bookclerk_plugin_abi

Authoritative Bookclerk plugin ABI (`api_version` 3).

Version 3 is the capability-driven object-capability ABI (Cap'n Proto RPC,
transferred byte streams). The guest root is [`PluginWorker`]:
`describe()` advertises typed [`PluginCapabilities`](generated.md#bookclerk_plugin_abigeneratedplugincapabilities), then
`open(invocation, bindings)` returns the exported [`Entrypoints`](roles.md#bookclerk_plugin_abirolesentrypoints)
(`storefront`, `storage`, `databaseAdapter`, `remoteLibrary`, `cli`,
`oidc`, plus the `eventConsumer` / `jobRunner` triggers). Host-granted
[`Bindings`](roles.md#bookclerk_plugin_abirolesbindings) carry `CONFIG` / `SECRETS`, the `EVENTS` publisher, and named
`[[databases]]` sessions — the Workers `env` idiom.

# Audience

- **Guest authors** — implement [`PluginWorker`](roles.md#bookclerk_plugin_abirolespluginworker) and the entrypoint traits
  against these DTOs (via `bookclerk-plugin-sdk`, `@bookclerk/plugin-sdk`,
  or a language binding generated from the same schema).
- **Host / SDK maintainers** — drive entrypoint capabilities, seal
  credentials, and upsert library rows without depending on store-specific
  crates.

Product narrative (jail, consent, install layout) lives in
[`docs/plugins.md`](https://github.com/bookclerk/bookclerk/blob/main/docs/plugins.md).
This crate is the typed wire contract only.

# Schema

The Cap'n Proto schema at
[`schema/plugin.capnp`](https://github.com/bookclerk/bookclerk/blob/main/crates/bookclerk-plugin-abi/schema/plugin.capnp)
is the single source of truth: RPC interfaces, product constants, database
enums, and the "Typed method payloads" section. Types here are the Rust
projection; the TypeScript and
Python SDK projections are generated from the same schema by
`scripts/gen-plugin-abi.py`, which also drift-checks this crate. Wire DTO
fields serialize as **camelCase**.

Install manifests validate against [`PLUGIN_TOML_SCHEMA_JSON`](bookclerk_plugin_abi.md#bookclerk_plugin_abiplugin_toml_schema_json)
(`schema/plugin-toml.json`).

# Versioning

[`PRODUCT_API_VERSION`](limits.md#bookclerk_plugin_abilimitsproduct_api_version) is `3` (capability-driven Cap'n Proto / Workers
RPC). Product spawn requires `plugin.toml` `api_version = 3`. There is no
`protocol` key.

# Modules

| Module | Contents |
| --- | --- |
| [`methods`](methods.md) | Capability / consent method name constants (`login`, `event`, …) |
| Crate-root DTOs | [`PluginDescribe`](rpc_types.md#bookclerk_plugin_abirpc_typesplugindescribe), health, CLI, destination objects |
| [`kind`](kind.md) | Kind-specific DTOs (source / integration / output) |
| [`settings_schema`](settings_schema.md) | `[[settings]]` operator settings schema shared by manifest and host |
| [`package_version`](package_version.md) | `plugin.toml` `version` parsing and ordering for schema pinning |
| [`db`](db.md) | Host-private database connect params (feature `host`) |
| [`error`](error.md) | [`PluginError`](error.md#bookclerk_plugin_abierrorpluginerror) / [`PluginErrorCode`](error.md#bookclerk_plugin_abierrorpluginerrorcode) |
| [`guest_sql`](guest_sql.md) | SQL-v1 grammar / guest admission |
| [`sql_desugar`](sql_desugar.md) | Host-only semantic desugars (`NULLS`, `NULLIF`) |
| [`plugin_capnp`](plugin_capnp.md) | Generated Cap'n Proto RPC interfaces |

## Modules

### [`bookclerk_plugin_abi`](bookclerk_plugin_abi.md)

*1 constant, 16 modules*

### [`backup_ops`](backup_ops.md)

*1 trait, 1 type alias*

### [`db`](db.md)

*1 constant, 1 enum, 4 functions*

### [`db_execute`](db_execute.md)

*10 structs, 25 constants, 3 enums, 5 functions*

### [`db_rpc`](db_rpc.md)

*9 functions*

### [`db_value`](db_value.md)

*2 enums, 2 functions*

### [`error`](error.md)

*1 enum, 1 struct, 1 type alias*

### [`features`](features.md)

*1 function, 1 struct, 3 constants*

### [`generated`](generated.md)

*1 constant, 166 functions, 60 structs, 8 enums*

### [`guest_sql`](guest_sql.md)

*1 constant, 2 structs, 9 functions*

### [`host_envelope`](host_envelope.md)

*4 structs*

### [`host_roles`](host_roles.md)

*2 traits*

### [`host_rpc`](host_rpc.md)

*1 struct*

### [`jobs`](jobs.md)

*2 functions, 2 structs*

### [`json_bytes`](json_bytes.md)

*2 functions, 2 modules*

### [`json_bytes::b64`](json_bytes/b64.md)

*2 functions*

### [`json_bytes::opt_b64`](json_bytes/opt_b64.md)

*2 functions*

### [`kind`](kind.md)

*3 structs*

### [`limits`](limits.md)

*1 struct, 11 constants*

### [`methods`](methods.md)

*1 constant, 35 modules*

### [`methods::authenticate_user`](methods/authenticate_user.md)

*1 constant*

### [`methods::catalog_detail`](methods/catalog_detail.md)

*1 constant*

### [`methods::clients`](methods/clients.md)

*1 constant*

### [`methods::copy`](methods/copy.md)

*1 constant*

### [`methods::delete`](methods/delete.md)

*1 constant*

### [`methods::describe`](methods/describe.md)

*1 constant*

### [`methods::diagnose`](methods/diagnose.md)

*1 constant*

### [`methods::event`](methods/event.md)

*1 constant*

### [`methods::exists`](methods/exists.md)

*1 constant*

### [`methods::expand_candidates`](methods/expand_candidates.md)

*1 constant*

### [`methods::fetch_title`](methods/fetch_title.md)

*1 constant*

### [`methods::get`](methods/get.md)

*1 constant*

### [`methods::health`](methods/health.md)

*1 constant*

### [`methods::invoke`](methods/invoke.md)

*1 constant*

### [`methods::list`](methods/list.md)

*1 constant*

### [`methods::list_accounts`](methods/list_accounts.md)

*1 constant*

### [`methods::list_deals`](methods/list_deals.md)

*1 constant*

### [`methods::login`](methods/login.md)

*1 constant*

### [`methods::login_complete`](methods/login_complete.md)

*1 constant*

### [`methods::login_start`](methods/login_start.md)

*1 constant*

### [`methods::poll_events`](methods/poll_events.md)

*1 constant*

### [`methods::publish`](methods/publish.md)

*1 constant*

### [`methods::purchase_hint`](methods/purchase_hint.md)

*1 constant*

### [`methods::put`](methods/put.md)

*1 constant*

### [`methods::queue`](methods/queue.md)

*1 constant*

### [`methods::scan`](methods/scan.md)

*1 constant*

### [`methods::scan_library`](methods/scan_library.md)

*1 constant*

### [`methods::search_catalog`](methods/search_catalog.md)

*1 constant*

### [`methods::send`](methods/send.md)

*1 constant*

### [`methods::send_batch`](methods/send_batch.md)

*1 constant*

### [`methods::shutdown`](methods/shutdown.md)

*1 constant*

### [`methods::start`](methods/start.md)

*1 constant*

### [`methods::stop`](methods/stop.md)

*1 constant*

### [`methods::sync_listening`](methods/sync_listening.md)

*1 constant*

### [`package_version`](package_version.md)

*2 functions*

### [`plugin_capnp`](plugin_capnp.md)

*13 enums, 17 constants, 197 modules*

### [`plugin_capnp::account_credential`](plugin_capnp/account_credential.md)

*4 structs*

### [`plugin_capnp::adapter_database_session`](plugin_capnp/adapter_database_session.md)

*1 trait, 20 modules, 20 type aliases, 3 structs*

### [`plugin_capnp::adapter_database_session::assert_restore_constraints_params`](plugin_capnp/adapter_database_session/assert_restore_constraints_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::assert_restore_constraints_results`](plugin_capnp/adapter_database_session/assert_restore_constraints_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::bootstrap_params`](plugin_capnp/adapter_database_session/bootstrap_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::bootstrap_results`](plugin_capnp/adapter_database_session/bootstrap_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::capabilities_params`](plugin_capnp/adapter_database_session/capabilities_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::capabilities_results`](plugin_capnp/adapter_database_session/capabilities_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::close_params`](plugin_capnp/adapter_database_session/close_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::close_results`](plugin_capnp/adapter_database_session/close_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::drop_user_relations_params`](plugin_capnp/adapter_database_session/drop_user_relations_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::drop_user_relations_results`](plugin_capnp/adapter_database_session/drop_user_relations_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::execute_params`](plugin_capnp/adapter_database_session/execute_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::execute_results`](plugin_capnp/adapter_database_session/execute_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::export_identity_params`](plugin_capnp/adapter_database_session/export_identity_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::export_identity_results`](plugin_capnp/adapter_database_session/export_identity_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::import_identity_params`](plugin_capnp/adapter_database_session/import_identity_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::import_identity_results`](plugin_capnp/adapter_database_session/import_identity_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::list_user_relations_params`](plugin_capnp/adapter_database_session/list_user_relations_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::list_user_relations_results`](plugin_capnp/adapter_database_session/list_user_relations_results.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::prepare_unit_restore_params`](plugin_capnp/adapter_database_session/prepare_unit_restore_params.md)

*4 structs*

### [`plugin_capnp::adapter_database_session::prepare_unit_restore_results`](plugin_capnp/adapter_database_session/prepare_unit_restore_results.md)

*4 structs*

### [`plugin_capnp::adapter_execute_request`](plugin_capnp/adapter_execute_request.md)

*4 structs*

### [`plugin_capnp::adapter_receipt`](plugin_capnp/adapter_receipt.md)

*4 structs*

### [`plugin_capnp::adapter_session_reply`](plugin_capnp/adapter_session_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::adapter_statement`](plugin_capnp/adapter_statement.md)

*4 structs*

### [`plugin_capnp::authenticate_user_params`](plugin_capnp/authenticate_user_params.md)

*4 structs*

### [`plugin_capnp::bindings`](plugin_capnp/bindings.md)

*4 structs*

### [`plugin_capnp::brand`](plugin_capnp/brand.md)

*4 structs*

### [`plugin_capnp::byte_range`](plugin_capnp/byte_range.md)

*4 structs*

### [`plugin_capnp::byte_source`](plugin_capnp/byte_source.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::byte_source::pull_params`](plugin_capnp/byte_source/pull_params.md)

*4 structs*

### [`plugin_capnp::byte_source::pull_results`](plugin_capnp/byte_source/pull_results.md)

*4 structs*

### [`plugin_capnp::cancellation`](plugin_capnp/cancellation.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::cancellation::poll_params`](plugin_capnp/cancellation/poll_params.md)

*4 structs*

### [`plugin_capnp::cancellation::poll_results`](plugin_capnp/cancellation/poll_results.md)

*4 structs*

### [`plugin_capnp::cancelled_outcome`](plugin_capnp/cancelled_outcome.md)

*4 structs*

### [`plugin_capnp::catalog_detail`](plugin_capnp/catalog_detail.md)

*4 structs*

### [`plugin_capnp::catalog_detail_params`](plugin_capnp/catalog_detail_params.md)

*4 structs*

### [`plugin_capnp::catalog_detail_reply`](plugin_capnp/catalog_detail_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::catalog_hit`](plugin_capnp/catalog_hit.md)

*4 structs*

### [`plugin_capnp::catalog_hits`](plugin_capnp/catalog_hits.md)

*4 structs*

### [`plugin_capnp::catalog_hits_reply`](plugin_capnp/catalog_hits_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::chapter_marker`](plugin_capnp/chapter_marker.md)

*4 structs*

### [`plugin_capnp::cli_arg`](plugin_capnp/cli_arg.md)

*4 structs*

### [`plugin_capnp::cli_arg_spec`](plugin_capnp/cli_arg_spec.md)

*4 structs*

### [`plugin_capnp::cli_command_spec`](plugin_capnp/cli_command_spec.md)

*4 structs*

### [`plugin_capnp::cli_invoke_params`](plugin_capnp/cli_invoke_params.md)

*4 structs*

### [`plugin_capnp::cli_invoke_reply`](plugin_capnp/cli_invoke_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::cli_invoke_result`](plugin_capnp/cli_invoke_result.md)

*4 structs*

### [`plugin_capnp::cli_schema`](plugin_capnp/cli_schema.md)

*4 structs*

### [`plugin_capnp::cli_schema_reply`](plugin_capnp/cli_schema_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::column_reference`](plugin_capnp/column_reference.md)

*4 structs*

### [`plugin_capnp::completed_outcome`](plugin_capnp/completed_outcome.md)

*4 structs*

### [`plugin_capnp::config_option`](plugin_capnp/config_option.md)

*4 structs*

### [`plugin_capnp::config_option_value`](plugin_capnp/config_option_value.md)

*4 structs*

### [`plugin_capnp::content_source`](plugin_capnp/content_source.md)

*1 trait, 26 modules, 26 type aliases, 3 structs*

### [`plugin_capnp::content_source::catalog_detail_params`](plugin_capnp/content_source/catalog_detail_params.md)

*4 structs*

### [`plugin_capnp::content_source::catalog_detail_results`](plugin_capnp/content_source/catalog_detail_results.md)

*4 structs*

### [`plugin_capnp::content_source::diagnose_params`](plugin_capnp/content_source/diagnose_params.md)

*4 structs*

### [`plugin_capnp::content_source::diagnose_results`](plugin_capnp/content_source/diagnose_results.md)

*4 structs*

### [`plugin_capnp::content_source::expand_candidates_params`](plugin_capnp/content_source/expand_candidates_params.md)

*4 structs*

### [`plugin_capnp::content_source::expand_candidates_results`](plugin_capnp/content_source/expand_candidates_results.md)

*4 structs*

### [`plugin_capnp::content_source::fetch_title_params`](plugin_capnp/content_source/fetch_title_params.md)

*4 structs*

### [`plugin_capnp::content_source::fetch_title_results`](plugin_capnp/content_source/fetch_title_results.md)

*4 structs*

### [`plugin_capnp::content_source::health_params`](plugin_capnp/content_source/health_params.md)

*4 structs*

### [`plugin_capnp::content_source::health_results`](plugin_capnp/content_source/health_results.md)

*4 structs*

### [`plugin_capnp::content_source::list_accounts_params`](plugin_capnp/content_source/list_accounts_params.md)

*4 structs*

### [`plugin_capnp::content_source::list_accounts_results`](plugin_capnp/content_source/list_accounts_results.md)

*4 structs*

### [`plugin_capnp::content_source::list_deals_params`](plugin_capnp/content_source/list_deals_params.md)

*4 structs*

### [`plugin_capnp::content_source::list_deals_results`](plugin_capnp/content_source/list_deals_results.md)

*4 structs*

### [`plugin_capnp::content_source::login_complete_params`](plugin_capnp/content_source/login_complete_params.md)

*4 structs*

### [`plugin_capnp::content_source::login_complete_results`](plugin_capnp/content_source/login_complete_results.md)

*4 structs*

### [`plugin_capnp::content_source::login_params`](plugin_capnp/content_source/login_params.md)

*4 structs*

### [`plugin_capnp::content_source::login_results`](plugin_capnp/content_source/login_results.md)

*4 structs*

### [`plugin_capnp::content_source::login_start_params`](plugin_capnp/content_source/login_start_params.md)

*4 structs*

### [`plugin_capnp::content_source::login_start_results`](plugin_capnp/content_source/login_start_results.md)

*4 structs*

### [`plugin_capnp::content_source::purchase_hint_params`](plugin_capnp/content_source/purchase_hint_params.md)

*4 structs*

### [`plugin_capnp::content_source::purchase_hint_results`](plugin_capnp/content_source/purchase_hint_results.md)

*4 structs*

### [`plugin_capnp::content_source::scan_params`](plugin_capnp/content_source/scan_params.md)

*4 structs*

### [`plugin_capnp::content_source::scan_results`](plugin_capnp/content_source/scan_results.md)

*4 structs*

### [`plugin_capnp::content_source::search_catalog_params`](plugin_capnp/content_source/search_catalog_params.md)

*4 structs*

### [`plugin_capnp::content_source::search_catalog_results`](plugin_capnp/content_source/search_catalog_results.md)

*4 structs*

### [`plugin_capnp::copy_reply`](plugin_capnp/copy_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::copy_result`](plugin_capnp/copy_result.md)

*4 structs*

### [`plugin_capnp::create_table_schema`](plugin_capnp/create_table_schema.md)

*4 structs*

### [`plugin_capnp::database`](plugin_capnp/database.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_capnp::database::drop_unit_params`](plugin_capnp/database/drop_unit_params.md)

*4 structs*

### [`plugin_capnp::database::drop_unit_results`](plugin_capnp/database/drop_unit_results.md)

*4 structs*

### [`plugin_capnp::database::open_session_params`](plugin_capnp/database/open_session_params.md)

*4 structs*

### [`plugin_capnp::database::open_session_results`](plugin_capnp/database/open_session_results.md)

*4 structs*

### [`plugin_capnp::database_adapter_config`](plugin_capnp/database_adapter_config.md)

*4 structs*

### [`plugin_capnp::db_bootstrap`](plugin_capnp/db_bootstrap.md)

*4 structs*

### [`plugin_capnp::db_bootstrap_reply`](plugin_capnp/db_bootstrap_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::db_capabilities`](plugin_capnp/db_capabilities.md)

*4 structs*

### [`plugin_capnp::db_capabilities_reply`](plugin_capnp/db_capabilities_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::db_column`](plugin_capnp/db_column.md)

*4 structs*

### [`plugin_capnp::db_row`](plugin_capnp/db_row.md)

*4 structs*

### [`plugin_capnp::db_statement`](plugin_capnp/db_statement.md)

*4 structs*

### [`plugin_capnp::db_timing`](plugin_capnp/db_timing.md)

*4 structs*

### [`plugin_capnp::db_value`](plugin_capnp/db_value.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::describe_reply`](plugin_capnp/describe_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::destination`](plugin_capnp/destination.md)

*1 trait, 16 modules, 16 type aliases, 3 structs*

### [`plugin_capnp::destination::abort_stage_params`](plugin_capnp/destination/abort_stage_params.md)

*4 structs*

### [`plugin_capnp::destination::abort_stage_results`](plugin_capnp/destination/abort_stage_results.md)

*4 structs*

### [`plugin_capnp::destination::commit_params`](plugin_capnp/destination/commit_params.md)

*4 structs*

### [`plugin_capnp::destination::commit_results`](plugin_capnp/destination/commit_results.md)

*4 structs*

### [`plugin_capnp::destination::copy_params`](plugin_capnp/destination/copy_params.md)

*4 structs*

### [`plugin_capnp::destination::copy_results`](plugin_capnp/destination/copy_results.md)

*4 structs*

### [`plugin_capnp::destination::delete_params`](plugin_capnp/destination/delete_params.md)

*4 structs*

### [`plugin_capnp::destination::delete_results`](plugin_capnp/destination/delete_results.md)

*4 structs*

### [`plugin_capnp::destination::get_params`](plugin_capnp/destination/get_params.md)

*4 structs*

### [`plugin_capnp::destination::get_results`](plugin_capnp/destination/get_results.md)

*4 structs*

### [`plugin_capnp::destination::head_params`](plugin_capnp/destination/head_params.md)

*4 structs*

### [`plugin_capnp::destination::head_results`](plugin_capnp/destination/head_results.md)

*4 structs*

### [`plugin_capnp::destination::list_params`](plugin_capnp/destination/list_params.md)

*4 structs*

### [`plugin_capnp::destination::list_results`](plugin_capnp/destination/list_results.md)

*4 structs*

### [`plugin_capnp::destination::put_params`](plugin_capnp/destination/put_params.md)

*4 structs*

### [`plugin_capnp::destination::put_results`](plugin_capnp/destination/put_results.md)

*4 structs*

### [`plugin_capnp::diagnose_reply`](plugin_capnp/diagnose_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::diagnose_result`](plugin_capnp/diagnose_result.md)

*4 structs*

### [`plugin_capnp::domain_event`](plugin_capnp/domain_event.md)

*4 structs*

### [`plugin_capnp::empty_reply`](plugin_capnp/empty_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::entrypoints`](plugin_capnp/entrypoints.md)

*4 structs*

### [`plugin_capnp::entrypoints_reply`](plugin_capnp/entrypoints_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::event_ack`](plugin_capnp/event_ack.md)

*4 structs*

### [`plugin_capnp::event_batch`](plugin_capnp/event_batch.md)

*4 structs*

### [`plugin_capnp::event_batch_reply`](plugin_capnp/event_batch_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::event_consumer`](plugin_capnp/event_consumer.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::event_consumer::event_params`](plugin_capnp/event_consumer/event_params.md)

*4 structs*

### [`plugin_capnp::event_consumer::event_results`](plugin_capnp/event_consumer/event_results.md)

*4 structs*

### [`plugin_capnp::event_consumer_spec`](plugin_capnp/event_consumer_spec.md)

*4 structs*

### [`plugin_capnp::event_dead_letter`](plugin_capnp/event_dead_letter.md)

*4 structs*

### [`plugin_capnp::event_poll_reply`](plugin_capnp/event_poll_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::event_poll_result`](plugin_capnp/event_poll_result.md)

*4 structs*

### [`plugin_capnp::event_publisher`](plugin_capnp/event_publisher.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::event_publisher::publish_params`](plugin_capnp/event_publisher/publish_params.md)

*4 structs*

### [`plugin_capnp::event_publisher::publish_results`](plugin_capnp/event_publisher/publish_results.md)

*4 structs*

### [`plugin_capnp::event_reject`](plugin_capnp/event_reject.md)

*4 structs*

### [`plugin_capnp::event_result`](plugin_capnp/event_result.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::event_retry`](plugin_capnp/event_retry.md)

*4 structs*

### [`plugin_capnp::event_suspended`](plugin_capnp/event_suspended.md)

*4 structs*

### [`plugin_capnp::execute_reply`](plugin_capnp/execute_reply.md)

*4 structs*

### [`plugin_capnp::execute_request`](plugin_capnp/execute_request.md)

*4 structs*

### [`plugin_capnp::execute_result_reply`](plugin_capnp/execute_result_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::expand_candidates_params`](plugin_capnp/expand_candidates_params.md)

*4 structs*

### [`plugin_capnp::extensible_config`](plugin_capnp/extensible_config.md)

*4 structs*

### [`plugin_capnp::external_user`](plugin_capnp/external_user.md)

*4 structs*

### [`plugin_capnp::external_user_reply`](plugin_capnp/external_user_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::fetch_options`](plugin_capnp/fetch_options.md)

*4 structs*

### [`plugin_capnp::fetch_title_params`](plugin_capnp/fetch_title_params.md)

*4 structs*

### [`plugin_capnp::fetch_title_reply`](plugin_capnp/fetch_title_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::foreign_key_constraint`](plugin_capnp/foreign_key_constraint.md)

*4 structs*

### [`plugin_capnp::get_ok`](plugin_capnp/get_ok.md)

*4 structs*

### [`plugin_capnp::get_reply`](plugin_capnp/get_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::guest_database`](plugin_capnp/guest_database.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_capnp::guest_database::close_params`](plugin_capnp/guest_database/close_params.md)

*4 structs*

### [`plugin_capnp::guest_database::close_results`](plugin_capnp/guest_database/close_results.md)

*4 structs*

### [`plugin_capnp::guest_database::execute_params`](plugin_capnp/guest_database/execute_params.md)

*4 structs*

### [`plugin_capnp::guest_database::execute_results`](plugin_capnp/guest_database/execute_results.md)

*4 structs*

### [`plugin_capnp::guest_database_reply`](plugin_capnp/guest_database_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::handle_reply`](plugin_capnp/handle_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::head_ok`](plugin_capnp/head_ok.md)

*4 structs*

### [`plugin_capnp::head_reply`](plugin_capnp/head_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::health_ok`](plugin_capnp/health_ok.md)

*4 structs*

### [`plugin_capnp::health_reply`](plugin_capnp/health_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::identity_export_reply`](plugin_capnp/identity_export_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::identity_high_water`](plugin_capnp/identity_high_water.md)

*4 structs*

### [`plugin_capnp::integer_arith_site`](plugin_capnp/integer_arith_site.md)

*4 structs*

### [`plugin_capnp::invocation`](plugin_capnp/invocation.md)

*4 structs*

### [`plugin_capnp::job_controller`](plugin_capnp/job_controller.md)

*4 structs*

### [`plugin_capnp::job_invocation`](plugin_capnp/job_invocation.md)

*4 structs*

### [`plugin_capnp::job_outcome`](plugin_capnp/job_outcome.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::job_runner`](plugin_capnp/job_runner.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::job_runner::job_params`](plugin_capnp/job_runner/job_params.md)

*4 structs*

### [`plugin_capnp::job_runner::job_results`](plugin_capnp/job_runner/job_results.md)

*4 structs*

### [`plugin_capnp::json_enum`](plugin_capnp/json_enum.md)

*1 constant, 1 function*

### [`plugin_capnp::kv_delete_ok`](plugin_capnp/kv_delete_ok.md)

*4 structs*

### [`plugin_capnp::kv_delete_reply`](plugin_capnp/kv_delete_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::kv_get_ok`](plugin_capnp/kv_get_ok.md)

*4 structs*

### [`plugin_capnp::kv_get_reply`](plugin_capnp/kv_get_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::kv_list_key`](plugin_capnp/kv_list_key.md)

*4 structs*

### [`plugin_capnp::kv_list_ok`](plugin_capnp/kv_list_ok.md)

*4 structs*

### [`plugin_capnp::kv_list_options`](plugin_capnp/kv_list_options.md)

*4 structs*

### [`plugin_capnp::kv_list_reply`](plugin_capnp/kv_list_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::kv_namespace`](plugin_capnp/kv_namespace.md)

*1 trait, 3 structs, 8 modules, 8 type aliases*

### [`plugin_capnp::kv_namespace::delete_params`](plugin_capnp/kv_namespace/delete_params.md)

*4 structs*

### [`plugin_capnp::kv_namespace::delete_results`](plugin_capnp/kv_namespace/delete_results.md)

*4 structs*

### [`plugin_capnp::kv_namespace::get_params`](plugin_capnp/kv_namespace/get_params.md)

*4 structs*

### [`plugin_capnp::kv_namespace::get_results`](plugin_capnp/kv_namespace/get_results.md)

*4 structs*

### [`plugin_capnp::kv_namespace::list_params`](plugin_capnp/kv_namespace/list_params.md)

*4 structs*

### [`plugin_capnp::kv_namespace::list_results`](plugin_capnp/kv_namespace/list_results.md)

*4 structs*

### [`plugin_capnp::kv_namespace::put_params`](plugin_capnp/kv_namespace/put_params.md)

*4 structs*

### [`plugin_capnp::kv_namespace::put_results`](plugin_capnp/kv_namespace/put_results.md)

*4 structs*

### [`plugin_capnp::kv_put_ok`](plugin_capnp/kv_put_ok.md)

*4 structs*

### [`plugin_capnp::kv_put_options`](plugin_capnp/kv_put_options.md)

*4 structs*

### [`plugin_capnp::kv_put_reply`](plugin_capnp/kv_put_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::list_deals_params`](plugin_capnp/list_deals_params.md)

*4 structs*

### [`plugin_capnp::list_options`](plugin_capnp/list_options.md)

*4 structs*

### [`plugin_capnp::list_page`](plugin_capnp/list_page.md)

*4 structs*

### [`plugin_capnp::list_reply`](plugin_capnp/list_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::listening_progress`](plugin_capnp/listening_progress.md)

*4 structs*

### [`plugin_capnp::login_complete_params`](plugin_capnp/login_complete_params.md)

*4 structs*

### [`plugin_capnp::login_params`](plugin_capnp/login_params.md)

*4 structs*

### [`plugin_capnp::login_reply`](plugin_capnp/login_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::login_result`](plugin_capnp/login_result.md)

*4 structs*

### [`plugin_capnp::login_start_reply`](plugin_capnp/login_start_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::login_start_result`](plugin_capnp/login_start_result.md)

*4 structs*

### [`plugin_capnp::named_database`](plugin_capnp/named_database.md)

*4 structs*

### [`plugin_capnp::named_kv`](plugin_capnp/named_kv.md)

*4 structs*

### [`plugin_capnp::named_queue`](plugin_capnp/named_queue.md)

*4 structs*

### [`plugin_capnp::named_sql_type`](plugin_capnp/named_sql_type.md)

*4 structs*

### [`plugin_capnp::object_info`](plugin_capnp/object_info.md)

*4 structs*

### [`plugin_capnp::object_metadata`](plugin_capnp/object_metadata.md)

*4 structs*

### [`plugin_capnp::oidc`](plugin_capnp/oidc.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_capnp::oidc::authenticate_user_params`](plugin_capnp/oidc/authenticate_user_params.md)

*4 structs*

### [`plugin_capnp::oidc::authenticate_user_results`](plugin_capnp/oidc/authenticate_user_results.md)

*4 structs*

### [`plugin_capnp::oidc::clients_params`](plugin_capnp/oidc/clients_params.md)

*4 structs*

### [`plugin_capnp::oidc::clients_results`](plugin_capnp/oidc/clients_results.md)

*4 structs*

### [`plugin_capnp::oidc_client_template`](plugin_capnp/oidc_client_template.md)

*4 structs*

### [`plugin_capnp::oidc_clients_ok`](plugin_capnp/oidc_clients_ok.md)

*4 structs*

### [`plugin_capnp::oidc_clients_reply`](plugin_capnp/oidc_clients_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::open_ok`](plugin_capnp/open_ok.md)

*4 structs*

### [`plugin_capnp::open_reply`](plugin_capnp/open_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::optional`](plugin_capnp/optional.md)

*1 constant, 1 function*

### [`plugin_capnp::optional_column_reference`](plugin_capnp/optional_column_reference.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::physical_access`](plugin_capnp/physical_access.md)

*4 structs*

### [`plugin_capnp::plain_fetch`](plugin_capnp/plain_fetch.md)

*4 structs*

### [`plugin_capnp::plain_part`](plugin_capnp/plain_part.md)

*4 structs*

### [`plugin_capnp::plugin_capabilities`](plugin_capnp/plugin_capabilities.md)

*4 structs*

### [`plugin_capnp::plugin_cli`](plugin_capnp/plugin_cli.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_capnp::plugin_cli::describe_params`](plugin_capnp/plugin_cli/describe_params.md)

*4 structs*

### [`plugin_capnp::plugin_cli::describe_results`](plugin_capnp/plugin_cli/describe_results.md)

*4 structs*

### [`plugin_capnp::plugin_cli::invoke_params`](plugin_capnp/plugin_cli/invoke_params.md)

*4 structs*

### [`plugin_capnp::plugin_cli::invoke_results`](plugin_capnp/plugin_cli/invoke_results.md)

*4 structs*

### [`plugin_capnp::plugin_describe`](plugin_capnp/plugin_describe.md)

*4 structs*

### [`plugin_capnp::plugin_error`](plugin_capnp/plugin_error.md)

*4 structs*

### [`plugin_capnp::plugin_event`](plugin_capnp/plugin_event.md)

*4 structs*

### [`plugin_capnp::plugin_migration`](plugin_capnp/plugin_migration.md)

*4 structs*

### [`plugin_capnp::plugin_migration_op`](plugin_capnp/plugin_migration_op.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::plugin_migrations_ok`](plugin_capnp/plugin_migrations_ok.md)

*4 structs*

### [`plugin_capnp::plugin_migrations_reply`](plugin_capnp/plugin_migrations_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::plugin_worker`](plugin_capnp/plugin_worker.md)

*1 trait, 3 structs, 8 modules, 8 type aliases*

### [`plugin_capnp::plugin_worker::database_migrations_params`](plugin_capnp/plugin_worker/database_migrations_params.md)

*4 structs*

### [`plugin_capnp::plugin_worker::database_migrations_results`](plugin_capnp/plugin_worker/database_migrations_results.md)

*4 structs*

### [`plugin_capnp::plugin_worker::describe_params`](plugin_capnp/plugin_worker/describe_params.md)

*4 structs*

### [`plugin_capnp::plugin_worker::describe_results`](plugin_capnp/plugin_worker/describe_results.md)

*4 structs*

### [`plugin_capnp::plugin_worker::open_params`](plugin_capnp/plugin_worker/open_params.md)

*4 structs*

### [`plugin_capnp::plugin_worker::open_results`](plugin_capnp/plugin_worker/open_results.md)

*4 structs*

### [`plugin_capnp::plugin_worker::shutdown_params`](plugin_capnp/plugin_worker/shutdown_params.md)

*4 structs*

### [`plugin_capnp::plugin_worker::shutdown_results`](plugin_capnp/plugin_worker/shutdown_results.md)

*4 structs*

### [`plugin_capnp::progress_sink`](plugin_capnp/progress_sink.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::progress_sink::report_params`](plugin_capnp/progress_sink/report_params.md)

*4 structs*

### [`plugin_capnp::progress_sink::report_results`](plugin_capnp/progress_sink/report_results.md)

*4 structs*

### [`plugin_capnp::publish_ok`](plugin_capnp/publish_ok.md)

*4 structs*

### [`plugin_capnp::publish_reply`](plugin_capnp/publish_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::pull_ok`](plugin_capnp/pull_ok.md)

*4 structs*

### [`plugin_capnp::pull_reply`](plugin_capnp/pull_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::purchase_hint`](plugin_capnp/purchase_hint.md)

*4 structs*

### [`plugin_capnp::purchase_hint_params`](plugin_capnp/purchase_hint_params.md)

*4 structs*

### [`plugin_capnp::purchase_hint_reply`](plugin_capnp/purchase_hint_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::purchase_hint_result`](plugin_capnp/purchase_hint_result.md)

*4 structs*

### [`plugin_capnp::put_reply`](plugin_capnp/put_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::put_result`](plugin_capnp/put_result.md)

*4 structs*

### [`plugin_capnp::queue_batch`](plugin_capnp/queue_batch.md)

*4 structs*

### [`plugin_capnp::queue_batch_ok`](plugin_capnp/queue_batch_ok.md)

*4 structs*

### [`plugin_capnp::queue_batch_reply`](plugin_capnp/queue_batch_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::queue_consumer`](plugin_capnp/queue_consumer.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::queue_consumer::queue_params`](plugin_capnp/queue_consumer/queue_params.md)

*4 structs*

### [`plugin_capnp::queue_consumer::queue_results`](plugin_capnp/queue_consumer/queue_results.md)

*4 structs*

### [`plugin_capnp::queue_message`](plugin_capnp/queue_message.md)

*4 structs*

### [`plugin_capnp::queue_producer`](plugin_capnp/queue_producer.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_capnp::queue_producer::send_batch_params`](plugin_capnp/queue_producer/send_batch_params.md)

*4 structs*

### [`plugin_capnp::queue_producer::send_batch_results`](plugin_capnp/queue_producer/send_batch_results.md)

*4 structs*

### [`plugin_capnp::queue_producer::send_params`](plugin_capnp/queue_producer/send_params.md)

*4 structs*

### [`plugin_capnp::queue_producer::send_results`](plugin_capnp/queue_producer/send_results.md)

*4 structs*

### [`plugin_capnp::queue_result`](plugin_capnp/queue_result.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::queue_retry`](plugin_capnp/queue_retry.md)

*4 structs*

### [`plugin_capnp::queue_send_message`](plugin_capnp/queue_send_message.md)

*4 structs*

### [`plugin_capnp::queue_send_ok`](plugin_capnp/queue_send_ok.md)

*4 structs*

### [`plugin_capnp::queue_send_reply`](plugin_capnp/queue_send_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::read_options`](plugin_capnp/read_options.md)

*4 structs*

### [`plugin_capnp::rejected_outcome`](plugin_capnp/rejected_outcome.md)

*4 structs*

### [`plugin_capnp::remote_library`](plugin_capnp/remote_library.md)

*1 trait, 14 modules, 14 type aliases, 3 structs*

### [`plugin_capnp::remote_library::diagnose_params`](plugin_capnp/remote_library/diagnose_params.md)

*4 structs*

### [`plugin_capnp::remote_library::diagnose_results`](plugin_capnp/remote_library/diagnose_results.md)

*4 structs*

### [`plugin_capnp::remote_library::health_params`](plugin_capnp/remote_library/health_params.md)

*4 structs*

### [`plugin_capnp::remote_library::health_results`](plugin_capnp/remote_library/health_results.md)

*4 structs*

### [`plugin_capnp::remote_library::poll_events_params`](plugin_capnp/remote_library/poll_events_params.md)

*4 structs*

### [`plugin_capnp::remote_library::poll_events_results`](plugin_capnp/remote_library/poll_events_results.md)

*4 structs*

### [`plugin_capnp::remote_library::scan_library_params`](plugin_capnp/remote_library/scan_library_params.md)

*4 structs*

### [`plugin_capnp::remote_library::scan_library_results`](plugin_capnp/remote_library/scan_library_results.md)

*4 structs*

### [`plugin_capnp::remote_library::start_params`](plugin_capnp/remote_library/start_params.md)

*4 structs*

### [`plugin_capnp::remote_library::start_results`](plugin_capnp/remote_library/start_results.md)

*4 structs*

### [`plugin_capnp::remote_library::stop_params`](plugin_capnp/remote_library/stop_params.md)

*4 structs*

### [`plugin_capnp::remote_library::stop_results`](plugin_capnp/remote_library/stop_results.md)

*4 structs*

### [`plugin_capnp::remote_library::sync_listening_params`](plugin_capnp/remote_library/sync_listening_params.md)

*4 structs*

### [`plugin_capnp::remote_library::sync_listening_results`](plugin_capnp/remote_library/sync_listening_results.md)

*4 structs*

### [`plugin_capnp::resolved_assignment`](plugin_capnp/resolved_assignment.md)

*4 structs*

### [`plugin_capnp::resolved_statement`](plugin_capnp/resolved_statement.md)

*4 structs*

### [`plugin_capnp::retryable_outcome`](plugin_capnp/retryable_outcome.md)

*4 structs*

### [`plugin_capnp::scalar_limits`](plugin_capnp/scalar_limits.md)

*4 structs*

### [`plugin_capnp::scan_book`](plugin_capnp/scan_book.md)

*4 structs*

### [`plugin_capnp::scan_library_params`](plugin_capnp/scan_library_params.md)

*4 structs*

### [`plugin_capnp::scan_params`](plugin_capnp/scan_params.md)

*4 structs*

### [`plugin_capnp::scan_reply`](plugin_capnp/scan_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::scan_summary`](plugin_capnp/scan_summary.md)

*4 structs*

### [`plugin_capnp::schema_action`](plugin_capnp/schema_action.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::schema_create`](plugin_capnp/schema_create.md)

*4 structs*

### [`plugin_capnp::search_catalog_params`](plugin_capnp/search_catalog_params.md)

*4 structs*

### [`plugin_capnp::source`](plugin_capnp/source.md)

*1 trait, 2 modules, 2 type aliases, 3 structs*

### [`plugin_capnp::source::open_params`](plugin_capnp/source/open_params.md)

*4 structs*

### [`plugin_capnp::source::open_results`](plugin_capnp/source/open_results.md)

*4 structs*

### [`plugin_capnp::source_account`](plugin_capnp/source_account.md)

*4 structs*

### [`plugin_capnp::source_accounts`](plugin_capnp/source_accounts.md)

*4 structs*

### [`plugin_capnp::source_accounts_reply`](plugin_capnp/source_accounts_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::sql_span`](plugin_capnp/sql_span.md)

*4 structs*

### [`plugin_capnp::statement_result`](plugin_capnp/statement_result.md)

*4 structs*

### [`plugin_capnp::suspended_outcome`](plugin_capnp/suspended_outcome.md)

*4 structs*

### [`plugin_capnp::sync_listening_reply`](plugin_capnp/sync_listening_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::sync_listening_result`](plugin_capnp/sync_listening_result.md)

*4 structs*

### [`plugin_capnp::table_constraint`](plugin_capnp/table_constraint.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::text_collate_site`](plugin_capnp/text_collate_site.md)

*4 structs*

### [`plugin_capnp::user_relations_reply`](plugin_capnp/user_relations_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_capnp::write_options`](plugin_capnp/write_options.md)

*4 structs*

### [`plugin_host_capnp`](plugin_host_capnp.md)

*3 modules*

### [`plugin_host_capnp::adapter_transaction`](plugin_host_capnp/adapter_transaction.md)

*1 trait, 18 modules, 18 type aliases, 3 structs*

### [`plugin_host_capnp::adapter_transaction::assert_restore_constraints_params`](plugin_host_capnp/adapter_transaction/assert_restore_constraints_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::assert_restore_constraints_results`](plugin_host_capnp/adapter_transaction/assert_restore_constraints_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::commit_params`](plugin_host_capnp/adapter_transaction/commit_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::commit_results`](plugin_host_capnp/adapter_transaction/commit_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::drop_user_relations_params`](plugin_host_capnp/adapter_transaction/drop_user_relations_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::drop_user_relations_results`](plugin_host_capnp/adapter_transaction/drop_user_relations_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::execute_params`](plugin_host_capnp/adapter_transaction/execute_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::execute_results`](plugin_host_capnp/adapter_transaction/execute_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::export_identity_params`](plugin_host_capnp/adapter_transaction/export_identity_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::export_identity_results`](plugin_host_capnp/adapter_transaction/export_identity_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::import_identity_params`](plugin_host_capnp/adapter_transaction/import_identity_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::import_identity_results`](plugin_host_capnp/adapter_transaction/import_identity_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::list_user_relations_params`](plugin_host_capnp/adapter_transaction/list_user_relations_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::list_user_relations_results`](plugin_host_capnp/adapter_transaction/list_user_relations_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::prepare_unit_restore_params`](plugin_host_capnp/adapter_transaction/prepare_unit_restore_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::prepare_unit_restore_results`](plugin_host_capnp/adapter_transaction/prepare_unit_restore_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::rollback_params`](plugin_host_capnp/adapter_transaction/rollback_params.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction::rollback_results`](plugin_host_capnp/adapter_transaction/rollback_results.md)

*4 structs*

### [`plugin_host_capnp::adapter_transaction_reply`](plugin_host_capnp/adapter_transaction_reply.md)

*1 enum, 2 type aliases, 4 structs*

### [`plugin_host_capnp::host_adapter_database_session`](plugin_host_capnp/host_adapter_database_session.md)

*1 trait, 3 structs, 4 modules, 4 type aliases*

### [`plugin_host_capnp::host_adapter_database_session::begin_params`](plugin_host_capnp/host_adapter_database_session/begin_params.md)

*4 structs*

### [`plugin_host_capnp::host_adapter_database_session::begin_results`](plugin_host_capnp/host_adapter_database_session/begin_results.md)

*4 structs*

### [`plugin_host_capnp::host_adapter_database_session::execute_params`](plugin_host_capnp/host_adapter_database_session/execute_params.md)

*4 structs*

### [`plugin_host_capnp::host_adapter_database_session::execute_results`](plugin_host_capnp/host_adapter_database_session/execute_results.md)

*4 structs*

### [`plugin_migration`](plugin_migration.md)

*1 enum, 1 struct, 2 constants, 4 functions*

### [`roles`](roles.md)

*18 traits, 9 structs*

### [`rpc`](rpc.md)

*20 structs, 5 functions*

### [`rpc_types`](rpc_types.md)

*17 structs, 2 constants, 2 enums*

### [`settings_schema`](settings_schema.md)

*1 enum, 1 function, 4 constants, 4 structs*

### [`sql_desugar`](sql_desugar.md)

*2 functions*

### [`sql_overflow`](sql_overflow.md)

*1 enum, 1 function*

### [`sql_proof`](sql_proof.md)

*1 constant, 1 function, 2 enums, 6 structs*

### [`sql_text`](sql_text.md)

*1 enum, 1 struct, 14 functions, 9 constants*

### [`sql_types`](sql_types.md)

*2 enums, 30 functions, 4 structs, 9 constants*

