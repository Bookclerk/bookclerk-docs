# bookclerk_plugin_abi::plugin_capnp

Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).

Included at crate root because `capnpc` emits `crate::plugin_capnp` paths.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PluginErrorCode`](#pluginerrorcode) | enum |  |
| [`Entrypoint`](#entrypoint) | enum |  |
| [`PortalAuthMode`](#portalauthmode) | enum |  |
| [`CliArgKind`](#cliargkind) | enum |  |
| [`CatalogSort`](#catalogsort) | enum |  |
| [`CatalogField`](#catalogfield) | enum |  |
| [`Abridgement`](#abridgement) | enum |  |
| [`DbType`](#dbtype) | enum |  |
| [`DbStatementKind`](#dbstatementkind) | enum |  |
| [`DbResultSelection`](#dbresultselection) | enum |  |
| [`IsolationReq`](#isolationreq) | enum |  |
| [`ResolvedSqlType`](#resolvedsqltype) | enum |  |
| [`IntegerArithKind`](#integerarithkind) | enum |  |
| [`API_VERSION`](#api_version) | constant |  |
| [`MAX_SCALAR_BYTES`](#max_scalar_bytes) | constant |  |
| [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) | constant |  |
| [`MAX_LIST_PAGE`](#max_list_page) | constant |  |
| [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) | constant |  |
| [`MAX_IDENTIFIER_BYTES`](#max_identifier_bytes) | constant |  |
| [`MAX_CONFIG_PAYLOAD_BYTES`](#max_config_payload_bytes) | constant |  |
| [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes) | constant |  |
| [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) | constant |  |
| [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes) | constant |  |
| [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes) | constant |  |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant |  |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant |  |
| [`FEATURE_STREAMS`](#feature_streams) | constant |  |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant |  |

## Modules

- [`scalar_limits`](plugin_capnp/scalar_limits.md)
- [`plugin_error`](plugin_capnp/plugin_error.md)
- [`object_metadata`](plugin_capnp/object_metadata.md)
- [`object_info`](plugin_capnp/object_info.md)
- [`list_options`](plugin_capnp/list_options.md)
- [`list_page`](plugin_capnp/list_page.md)
- [`byte_range`](plugin_capnp/byte_range.md)
- [`read_options`](plugin_capnp/read_options.md)
- [`write_options`](plugin_capnp/write_options.md)
- [`put_result`](plugin_capnp/put_result.md)
- [`copy_result`](plugin_capnp/copy_result.md)
- [`plugin_describe`](plugin_capnp/plugin_describe.md)
- [`optional`](plugin_capnp/optional.md)
- [`oidc_client_template`](plugin_capnp/oidc_client_template.md)
- [`oidc_clients_ok`](plugin_capnp/oidc_clients_ok.md)
- [`oidc_clients_reply`](plugin_capnp/oidc_clients_reply.md)
- [`extensible_config`](plugin_capnp/extensible_config.md)
- [`bindings`](plugin_capnp/bindings.md)
- [`entrypoints`](plugin_capnp/entrypoints.md)
- [`entrypoints_reply`](plugin_capnp/entrypoints_reply.md)
- [`job_invocation`](plugin_capnp/job_invocation.md)
- [`completed_outcome`](plugin_capnp/completed_outcome.md)
- [`retryable_outcome`](plugin_capnp/retryable_outcome.md)
- [`rejected_outcome`](plugin_capnp/rejected_outcome.md)
- [`cancelled_outcome`](plugin_capnp/cancelled_outcome.md)
- [`suspended_outcome`](plugin_capnp/suspended_outcome.md)
- [`job_outcome`](plugin_capnp/job_outcome.md)
- [`domain_event`](plugin_capnp/domain_event.md)
- [`event_ack`](plugin_capnp/event_ack.md)
- [`event_retry`](plugin_capnp/event_retry.md)
- [`event_reject`](plugin_capnp/event_reject.md)
- [`event_dead_letter`](plugin_capnp/event_dead_letter.md)
- [`event_suspended`](plugin_capnp/event_suspended.md)
- [`event_result`](plugin_capnp/event_result.md)
- [`event_batch`](plugin_capnp/event_batch.md)
- [`event_batch_reply`](plugin_capnp/event_batch_reply.md)
- [`job_controller`](plugin_capnp/job_controller.md)
- [`head_ok`](plugin_capnp/head_ok.md)
- [`head_reply`](plugin_capnp/head_reply.md)
- [`list_reply`](plugin_capnp/list_reply.md)
- [`get_ok`](plugin_capnp/get_ok.md)
- [`get_reply`](plugin_capnp/get_reply.md)
- [`put_reply`](plugin_capnp/put_reply.md)
- [`copy_reply`](plugin_capnp/copy_reply.md)
- [`empty_reply`](plugin_capnp/empty_reply.md)
- [`pull_ok`](plugin_capnp/pull_ok.md)
- [`pull_reply`](plugin_capnp/pull_reply.md)
- [`open_ok`](plugin_capnp/open_ok.md)
- [`open_reply`](plugin_capnp/open_reply.md)
- [`describe_reply`](plugin_capnp/describe_reply.md)
- [`handle_reply`](plugin_capnp/handle_reply.md)
- [`health_ok`](plugin_capnp/health_ok.md)
- [`health_reply`](plugin_capnp/health_reply.md)
- [`adapter_session_reply`](plugin_capnp/adapter_session_reply.md)
- [`guest_database_reply`](plugin_capnp/guest_database_reply.md)
- [`byte_source`](plugin_capnp/byte_source.md)
- [`destination`](plugin_capnp/destination.md)
- [`source`](plugin_capnp/source.md)
- [`progress_sink`](plugin_capnp/progress_sink.md)
- [`cancellation`](plugin_capnp/cancellation.md)
- [`job_runner`](plugin_capnp/job_runner.md)
- [`event_consumer`](plugin_capnp/event_consumer.md)
- [`event_publisher`](plugin_capnp/event_publisher.md)
- [`named_database`](plugin_capnp/named_database.md)
- [`queue_consumer`](plugin_capnp/queue_consumer.md)
- [`queue_producer`](plugin_capnp/queue_producer.md)
- [`named_queue`](plugin_capnp/named_queue.md)
- [`kv_namespace`](plugin_capnp/kv_namespace.md)
- [`named_kv`](plugin_capnp/named_kv.md)
- [`content_source`](plugin_capnp/content_source.md)
- [`remote_library`](plugin_capnp/remote_library.md)
- [`plugin_cli`](plugin_capnp/plugin_cli.md)
- [`oidc`](plugin_capnp/oidc.md)
- [`json_enum`](plugin_capnp/json_enum.md)
- [`event_consumer_spec`](plugin_capnp/event_consumer_spec.md)
- [`plugin_capabilities`](plugin_capnp/plugin_capabilities.md)
- [`brand`](plugin_capnp/brand.md)
- [`config_option`](plugin_capnp/config_option.md)
- [`config_option_value`](plugin_capnp/config_option_value.md)
- [`cli_schema`](plugin_capnp/cli_schema.md)
- [`cli_command_spec`](plugin_capnp/cli_command_spec.md)
- [`cli_arg_spec`](plugin_capnp/cli_arg_spec.md)
- [`cli_arg`](plugin_capnp/cli_arg.md)
- [`cli_invoke_params`](plugin_capnp/cli_invoke_params.md)
- [`cli_invoke_result`](plugin_capnp/cli_invoke_result.md)
- [`cli_schema_reply`](plugin_capnp/cli_schema_reply.md)
- [`cli_invoke_reply`](plugin_capnp/cli_invoke_reply.md)
- [`database_adapter_config`](plugin_capnp/database_adapter_config.md)
- [`diagnose_result`](plugin_capnp/diagnose_result.md)
- [`diagnose_reply`](plugin_capnp/diagnose_reply.md)
- [`source_account`](plugin_capnp/source_account.md)
- [`login_params`](plugin_capnp/login_params.md)
- [`login_result`](plugin_capnp/login_result.md)
- [`login_start_result`](plugin_capnp/login_start_result.md)
- [`login_complete_params`](plugin_capnp/login_complete_params.md)
- [`login_reply`](plugin_capnp/login_reply.md)
- [`login_start_reply`](plugin_capnp/login_start_reply.md)
- [`account_credential`](plugin_capnp/account_credential.md)
- [`scan_params`](plugin_capnp/scan_params.md)
- [`scan_book`](plugin_capnp/scan_book.md)
- [`scan_summary`](plugin_capnp/scan_summary.md)
- [`scan_reply`](plugin_capnp/scan_reply.md)
- [`fetch_options`](plugin_capnp/fetch_options.md)
- [`fetch_title_params`](plugin_capnp/fetch_title_params.md)
- [`plain_part`](plugin_capnp/plain_part.md)
- [`chapter_marker`](plugin_capnp/chapter_marker.md)
- [`plain_fetch`](plugin_capnp/plain_fetch.md)
- [`fetch_title_reply`](plugin_capnp/fetch_title_reply.md)
- [`source_accounts`](plugin_capnp/source_accounts.md)
- [`source_accounts_reply`](plugin_capnp/source_accounts_reply.md)
- [`search_catalog_params`](plugin_capnp/search_catalog_params.md)
- [`expand_candidates_params`](plugin_capnp/expand_candidates_params.md)
- [`purchase_hint_params`](plugin_capnp/purchase_hint_params.md)
- [`list_deals_params`](plugin_capnp/list_deals_params.md)
- [`catalog_detail_params`](plugin_capnp/catalog_detail_params.md)
- [`revoke_account_params`](plugin_capnp/revoke_account_params.md)
- [`catalog_hit`](plugin_capnp/catalog_hit.md)
- [`catalog_hits`](plugin_capnp/catalog_hits.md)
- [`catalog_hits_reply`](plugin_capnp/catalog_hits_reply.md)
- [`catalog_detail`](plugin_capnp/catalog_detail.md)
- [`catalog_detail_reply`](plugin_capnp/catalog_detail_reply.md)
- [`purchase_hint`](plugin_capnp/purchase_hint.md)
- [`purchase_hint_result`](plugin_capnp/purchase_hint_result.md)
- [`purchase_hint_reply`](plugin_capnp/purchase_hint_reply.md)
- [`scan_library_params`](plugin_capnp/scan_library_params.md)
- [`authenticate_user_params`](plugin_capnp/authenticate_user_params.md)
- [`external_user`](plugin_capnp/external_user.md)
- [`external_user_reply`](plugin_capnp/external_user_reply.md)
- [`event_poll_result`](plugin_capnp/event_poll_result.md)
- [`event_poll_reply`](plugin_capnp/event_poll_reply.md)
- [`listening_progress`](plugin_capnp/listening_progress.md)
- [`sync_listening_result`](plugin_capnp/sync_listening_result.md)
- [`sync_listening_reply`](plugin_capnp/sync_listening_reply.md)
- [`plugin_event`](plugin_capnp/plugin_event.md)
- [`publish_ok`](plugin_capnp/publish_ok.md)
- [`publish_reply`](plugin_capnp/publish_reply.md)
- [`invocation`](plugin_capnp/invocation.md)
- [`queue_message`](plugin_capnp/queue_message.md)
- [`queue_batch`](plugin_capnp/queue_batch.md)
- [`queue_retry`](plugin_capnp/queue_retry.md)
- [`queue_result`](plugin_capnp/queue_result.md)
- [`queue_batch_ok`](plugin_capnp/queue_batch_ok.md)
- [`queue_batch_reply`](plugin_capnp/queue_batch_reply.md)
- [`queue_send_message`](plugin_capnp/queue_send_message.md)
- [`queue_send_ok`](plugin_capnp/queue_send_ok.md)
- [`queue_send_reply`](plugin_capnp/queue_send_reply.md)
- [`kv_put_options`](plugin_capnp/kv_put_options.md)
- [`kv_get_ok`](plugin_capnp/kv_get_ok.md)
- [`kv_get_reply`](plugin_capnp/kv_get_reply.md)
- [`kv_put_ok`](plugin_capnp/kv_put_ok.md)
- [`kv_delete_ok`](plugin_capnp/kv_delete_ok.md)
- [`kv_put_reply`](plugin_capnp/kv_put_reply.md)
- [`kv_delete_reply`](plugin_capnp/kv_delete_reply.md)
- [`kv_list_options`](plugin_capnp/kv_list_options.md)
- [`kv_list_key`](plugin_capnp/kv_list_key.md)
- [`kv_list_ok`](plugin_capnp/kv_list_ok.md)
- [`kv_list_reply`](plugin_capnp/kv_list_reply.md)
- [`db_value`](plugin_capnp/db_value.md)
- [`db_column`](plugin_capnp/db_column.md)
- [`db_row`](plugin_capnp/db_row.md)
- [`db_statement`](plugin_capnp/db_statement.md)
- [`execute_request`](plugin_capnp/execute_request.md)
- [`sql_span`](plugin_capnp/sql_span.md)
- [`text_collate_site`](plugin_capnp/text_collate_site.md)
- [`integer_arith_site`](plugin_capnp/integer_arith_site.md)
- [`physical_access`](plugin_capnp/physical_access.md)
- [`resolved_assignment`](plugin_capnp/resolved_assignment.md)
- [`named_sql_type`](plugin_capnp/named_sql_type.md)
- [`column_reference`](plugin_capnp/column_reference.md)
- [`optional_column_reference`](plugin_capnp/optional_column_reference.md)
- [`foreign_key_constraint`](plugin_capnp/foreign_key_constraint.md)
- [`table_constraint`](plugin_capnp/table_constraint.md)
- [`create_table_schema`](plugin_capnp/create_table_schema.md)
- [`schema_create`](plugin_capnp/schema_create.md)
- [`schema_action`](plugin_capnp/schema_action.md)
- [`resolved_statement`](plugin_capnp/resolved_statement.md)
- [`adapter_receipt`](plugin_capnp/adapter_receipt.md)
- [`adapter_statement`](plugin_capnp/adapter_statement.md)
- [`adapter_execute_request`](plugin_capnp/adapter_execute_request.md)
- [`statement_result`](plugin_capnp/statement_result.md)
- [`db_timing`](plugin_capnp/db_timing.md)
- [`execute_reply`](plugin_capnp/execute_reply.md)
- [`execute_result_reply`](plugin_capnp/execute_result_reply.md)
- [`db_capabilities`](plugin_capnp/db_capabilities.md)
- [`db_bootstrap_reply`](plugin_capnp/db_bootstrap_reply.md)
- [`db_bootstrap`](plugin_capnp/db_bootstrap.md)
- [`db_capabilities_reply`](plugin_capnp/db_capabilities_reply.md)
- [`database`](plugin_capnp/database.md)
- [`identity_high_water`](plugin_capnp/identity_high_water.md)
- [`identity_export_reply`](plugin_capnp/identity_export_reply.md)
- [`user_relations_reply`](plugin_capnp/user_relations_reply.md)
- [`adapter_database_session`](plugin_capnp/adapter_database_session.md)
- [`guest_database`](plugin_capnp/guest_database.md)
- [`plugin_migration_op`](plugin_capnp/plugin_migration_op.md)
- [`plugin_migration`](plugin_capnp/plugin_migration.md)
- [`plugin_migrations_ok`](plugin_capnp/plugin_migrations_ok.md)
- [`plugin_migrations_reply`](plugin_capnp/plugin_migrations_reply.md)
- [`plugin_worker`](plugin_capnp/plugin_worker.md)

## Enums

### PluginErrorCode

```rust
pub enum PluginErrorCode {
    InvalidParams = 0,
    Unauthorized = 1,
    Forbidden = 2,
    NotFound = 3,
    Unavailable = 4,
    Unsupported = 5,
    Internal = 6,
    PayloadTooLarge = 7,
    DeadlineExceeded = 8,
    InvalidCursor = 9,
    Cancelled = 10,
    Conflict = 11,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<PluginErrorCode>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### Entrypoint

```rust
pub enum Entrypoint {
    Storefront = 0,
    Storage = 1,
    DatabaseAdapter = 2,
    RemoteLibrary = 3,
    Cli = 4,
    Oidc = 5,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<Entrypoint>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### PortalAuthMode

```rust
pub enum PortalAuthMode {
    Unspecified = 0,
    Password = 1,
    Oauth = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<PortalAuthMode>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### CliArgKind

```rust
pub enum CliArgKind {
    String = 0,
    Bool = 1,
    Int = 2,
    Path = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<CliArgKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### CatalogSort

```rust
pub enum CatalogSort {
    Relevance = 0,
    Popularity = 1,
    Rating = 2,
    Title = 3,
    Author = 4,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<CatalogSort>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### CatalogField

```rust
pub enum CatalogField {
    Any = 0,
    Author = 1,
    Narrator = 2,
    Series = 3,
    Genre = 4,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<CatalogField>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### Abridgement

```rust
pub enum Abridgement {
    Unknown = 0,
    Unabridged = 1,
    Abridged = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<Abridgement>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### DbType

```rust
pub enum DbType {
    Unspecified = 0,
    Bool = 1,
    Int64 = 2,
    Float64 = 3,
    Text = 4,
    Bytes = 5,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<DbType>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### DbStatementKind

```rust
pub enum DbStatementKind {
    Execute = 0,
    Select = 1,
    Returning = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<DbStatementKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### DbResultSelection

```rust
pub enum DbResultSelection {
    Discard = 0,
    AffectedRows = 1,
    Rows = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<DbResultSelection>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### IsolationReq

```rust
pub enum IsolationReq {
    AtomicBatch = 0,
    NestedSavepoint = 1,
    ConsistentSnapshot = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<IsolationReq>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### ResolvedSqlType

```rust
pub enum ResolvedSqlType {
    Integer = 0,
    Real = 1,
    Text = 2,
    Blob = 3,
    Boolean = 4,
    Null = 5,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<ResolvedSqlType>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### IntegerArithKind

```rust
pub enum IntegerArithKind {
    Add = 0,
    Sub = 1,
    Mul = 2,
    Abs = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<IntegerArithKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

## Constants

### API_VERSION

```rust
pub const API_VERSION: u32 = 3;
```

### MAX_SCALAR_BYTES

```rust
pub const MAX_SCALAR_BYTES: u32 = 262144;
```

### MAX_STREAM_WINDOW_BYTES

```rust
pub const MAX_STREAM_WINDOW_BYTES: u32 = 1048576;
```

### MAX_LIST_PAGE

```rust
pub const MAX_LIST_PAGE: u32 = 256;
```

### MAX_CHECKPOINT_BYTES

```rust
pub const MAX_CHECKPOINT_BYTES: u32 = 65536;
```

### MAX_IDENTIFIER_BYTES

```rust
pub const MAX_IDENTIFIER_BYTES: u32 = 64;
```

### MAX_CONFIG_PAYLOAD_BYTES

```rust
pub const MAX_CONFIG_PAYLOAD_BYTES: u32 = 65536;
```

### MAX_EVENT_PAYLOAD_BYTES

```rust
pub const MAX_EVENT_PAYLOAD_BYTES: u32 = 65536;
```

### MAX_KV_VALUE_BYTES

```rust
pub const MAX_KV_VALUE_BYTES: u32 = 1048576;
```

### MAX_KV_KEY_BYTES

```rust
pub const MAX_KV_KEY_BYTES: u32 = 512;
```

### MAX_KV_METADATA_BYTES

```rust
pub const MAX_KV_METADATA_BYTES: u32 = 1024;
```

### MAX_PLUGIN_MIGRATION_OPS

```rust
pub const MAX_PLUGIN_MIGRATION_OPS: u32 = 256;
```

### MAX_PLUGIN_MIGRATION_TOTAL_OPS

```rust
pub const MAX_PLUGIN_MIGRATION_TOTAL_OPS: u32 = 2048;
```

### MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES

```rust
pub const MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES: u32 = 262144;
```

### FEATURE_SCALAR_LIMITS

```rust
pub const FEATURE_SCALAR_LIMITS: &str = "rpc.scalarLimits";
```

### FEATURE_STREAMS

```rust
pub const FEATURE_STREAMS: &str = "rpc.streams";
```

### FEATURE_STORAGE_COPY

```rust
pub const FEATURE_STORAGE_COPY: &str = "storage.copy";
```
