# bookclerk_plugin_abi::plugin_capnp

Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).

Included at crate root because `capnpc` emits `crate::plugin_capnp` paths.

| Item | Kind | Summary |
| --- | --- | --- |
| [`PluginErrorCode`](#pluginerrorcode) | enum |  |
| [`Entrypoint`](#entrypoint) | enum |  |
| [`PortalAuthMode`](#portalauthmode) | enum |  |
| [`CliArgKind`](#cliargkind) | enum |  |
| [`WorkKind`](#workkind) | enum |  |
| [`ExpressionKind`](#expressionkind) | enum |  |
| [`MediaKind`](#mediakind) | enum |  |
| [`IdentifierScheme`](#identifierscheme) | enum |  |
| [`CreditRole`](#creditrole) | enum |  |
| [`CreditLevel`](#creditlevel) | enum |  |
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
| [`MAX_CRON_BYTES`](#max_cron_bytes) | constant |  |
| [`MAX_CONFIG_PAYLOAD_BYTES`](#max_config_payload_bytes) | constant |  |
| [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes) | constant |  |
| [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) | constant |  |
| [`MAX_COVER_URLS`](#max_cover_urls) | constant |  |
| [`MAX_COVER_BYTES`](#max_cover_bytes) | constant |  |
| [`MAX_COVER_HEADER_BYTES`](#max_cover_header_bytes) | constant |  |
| [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes) | constant |  |
| [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes) | constant |  |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant |  |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant |  |
| [`FEATURE_STREAMS`](#feature_streams) | constant |  |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant |  |

## Modules

- [`scalar_limits`](scalar_limits.md)
- [`plugin_error`](plugin_error.md)
- [`object_metadata`](object_metadata.md)
- [`object_info`](object_info.md)
- [`list_options`](list_options.md)
- [`list_page`](list_page.md)
- [`byte_range`](byte_range.md)
- [`read_options`](read_options.md)
- [`write_options`](write_options.md)
- [`put_result`](put_result.md)
- [`copy_result`](copy_result.md)
- [`plugin_describe`](plugin_describe.md)
- [`optional`](optional.md)
- [`oidc_client_template`](oidc_client_template.md)
- [`oidc_clients_ok`](oidc_clients_ok.md)
- [`oidc_clients_reply`](oidc_clients_reply.md)
- [`extensible_config`](extensible_config.md)
- [`bindings`](bindings.md)
- [`entrypoints`](entrypoints.md)
- [`entrypoints_reply`](entrypoints_reply.md)
- [`job_invocation`](job_invocation.md)
- [`completed_outcome`](completed_outcome.md)
- [`retryable_outcome`](retryable_outcome.md)
- [`rejected_outcome`](rejected_outcome.md)
- [`cancelled_outcome`](cancelled_outcome.md)
- [`suspended_outcome`](suspended_outcome.md)
- [`job_outcome`](job_outcome.md)
- [`domain_event`](domain_event.md)
- [`event_ack`](event_ack.md)
- [`event_retry`](event_retry.md)
- [`event_reject`](event_reject.md)
- [`event_dead_letter`](event_dead_letter.md)
- [`event_suspended`](event_suspended.md)
- [`event_result`](event_result.md)
- [`event_batch`](event_batch.md)
- [`event_batch_reply`](event_batch_reply.md)
- [`job_controller`](job_controller.md)
- [`head_ok`](head_ok.md)
- [`head_reply`](head_reply.md)
- [`list_reply`](list_reply.md)
- [`get_ok`](get_ok.md)
- [`get_reply`](get_reply.md)
- [`put_reply`](put_reply.md)
- [`copy_reply`](copy_reply.md)
- [`empty_reply`](empty_reply.md)
- [`pull_ok`](pull_ok.md)
- [`pull_reply`](pull_reply.md)
- [`open_ok`](open_ok.md)
- [`open_reply`](open_reply.md)
- [`describe_reply`](describe_reply.md)
- [`handle_reply`](handle_reply.md)
- [`health_ok`](health_ok.md)
- [`health_reply`](health_reply.md)
- [`adapter_session_reply`](adapter_session_reply.md)
- [`guest_database_reply`](guest_database_reply.md)
- [`byte_source`](byte_source/README.md)
- [`destination`](destination/README.md)
- [`source`](source/README.md)
- [`progress_sink`](progress_sink/README.md)
- [`cancellation`](cancellation/README.md)
- [`job_runner`](job_runner/README.md)
- [`event_consumer`](event_consumer/README.md)
- [`event_publisher`](event_publisher/README.md)
- [`named_database`](named_database.md)
- [`scheduled_handler`](scheduled_handler/README.md)
- [`queue_consumer`](queue_consumer/README.md)
- [`queue_producer`](queue_producer/README.md)
- [`named_queue`](named_queue.md)
- [`kv_namespace`](kv_namespace/README.md)
- [`named_kv`](named_kv.md)
- [`content_source`](content_source/README.md)
- [`remote_library`](remote_library/README.md)
- [`plugin_cli`](plugin_cli/README.md)
- [`oidc`](oidc/README.md)
- [`metadata_provider`](metadata_provider/README.md)
- [`cover_fetcher`](cover_fetcher/README.md)
- [`json_enum`](json_enum.md)
- [`event_consumer_spec`](event_consumer_spec.md)
- [`plugin_capabilities`](plugin_capabilities.md)
- [`brand`](brand.md)
- [`config_option`](config_option.md)
- [`config_option_value`](config_option_value.md)
- [`cli_schema`](cli_schema.md)
- [`cli_command_spec`](cli_command_spec.md)
- [`cli_arg_spec`](cli_arg_spec.md)
- [`cli_arg`](cli_arg.md)
- [`cli_invoke_params`](cli_invoke_params.md)
- [`cli_invoke_result`](cli_invoke_result.md)
- [`cli_schema_reply`](cli_schema_reply.md)
- [`cli_invoke_reply`](cli_invoke_reply.md)
- [`database_adapter_config`](database_adapter_config.md)
- [`diagnose_result`](diagnose_result.md)
- [`diagnose_reply`](diagnose_reply.md)
- [`source_account`](source_account.md)
- [`login_params`](login_params.md)
- [`login_result`](login_result.md)
- [`login_start_result`](login_start_result.md)
- [`login_complete_params`](login_complete_params.md)
- [`login_reply`](login_reply.md)
- [`login_start_reply`](login_start_reply.md)
- [`account_credential`](account_credential.md)
- [`scan_params`](scan_params.md)
- [`identifier`](identifier.md)
- [`credit`](credit.md)
- [`series_entry`](series_entry.md)
- [`scan_book`](scan_book.md)
- [`scan_summary`](scan_summary.md)
- [`scan_reply`](scan_reply.md)
- [`fetch_options`](fetch_options.md)
- [`fetch_title_params`](fetch_title_params.md)
- [`plain_part`](plain_part.md)
- [`chapter_marker`](chapter_marker.md)
- [`plain_fetch`](plain_fetch.md)
- [`fetch_title_reply`](fetch_title_reply.md)
- [`source_accounts`](source_accounts.md)
- [`source_accounts_reply`](source_accounts_reply.md)
- [`search_catalog_params`](search_catalog_params.md)
- [`expand_candidates_params`](expand_candidates_params.md)
- [`purchase_hint_params`](purchase_hint_params.md)
- [`list_deals_params`](list_deals_params.md)
- [`catalog_detail_params`](catalog_detail_params.md)
- [`revoke_account_params`](revoke_account_params.md)
- [`catalog_hit`](catalog_hit.md)
- [`catalog_hits`](catalog_hits.md)
- [`catalog_hits_reply`](catalog_hits_reply.md)
- [`catalog_detail`](catalog_detail.md)
- [`catalog_detail_reply`](catalog_detail_reply.md)
- [`purchase_hint`](purchase_hint.md)
- [`purchase_hint_result`](purchase_hint_result.md)
- [`purchase_hint_reply`](purchase_hint_reply.md)
- [`scan_library_params`](scan_library_params.md)
- [`authenticate_user_params`](authenticate_user_params.md)
- [`external_user`](external_user.md)
- [`external_user_reply`](external_user_reply.md)
- [`event_poll_result`](event_poll_result.md)
- [`event_poll_reply`](event_poll_reply.md)
- [`listening_progress`](listening_progress.md)
- [`sync_listening_result`](sync_listening_result.md)
- [`sync_listening_reply`](sync_listening_reply.md)
- [`plugin_event`](plugin_event.md)
- [`publish_ok`](publish_ok.md)
- [`publish_reply`](publish_reply.md)
- [`invocation`](invocation.md)
- [`queue_message`](queue_message.md)
- [`queue_batch`](queue_batch.md)
- [`queue_retry`](queue_retry.md)
- [`queue_result`](queue_result.md)
- [`queue_batch_ok`](queue_batch_ok.md)
- [`queue_batch_reply`](queue_batch_reply.md)
- [`queue_send_message`](queue_send_message.md)
- [`queue_send_ok`](queue_send_ok.md)
- [`queue_send_reply`](queue_send_reply.md)
- [`kv_put_options`](kv_put_options.md)
- [`kv_get_ok`](kv_get_ok.md)
- [`kv_get_reply`](kv_get_reply.md)
- [`kv_put_ok`](kv_put_ok.md)
- [`kv_delete_ok`](kv_delete_ok.md)
- [`kv_put_reply`](kv_put_reply.md)
- [`kv_delete_reply`](kv_delete_reply.md)
- [`kv_list_options`](kv_list_options.md)
- [`kv_list_key`](kv_list_key.md)
- [`kv_list_ok`](kv_list_ok.md)
- [`kv_list_reply`](kv_list_reply.md)
- [`scheduled_controller`](scheduled_controller.md)
- [`metadata_lookup_params`](metadata_lookup_params.md)
- [`metadata_search_params`](metadata_search_params.md)
- [`metadata_chapters_params`](metadata_chapters_params.md)
- [`metadata_record`](metadata_record.md)
- [`metadata_records`](metadata_records.md)
- [`metadata_records_reply`](metadata_records_reply.md)
- [`metadata_chapter`](metadata_chapter.md)
- [`metadata_chapters`](metadata_chapters.md)
- [`metadata_chapters_reply`](metadata_chapters_reply.md)
- [`cover_fetch_params`](cover_fetch_params.md)
- [`cover_condition`](cover_condition.md)
- [`cover_image`](cover_image.md)
- [`cover_images`](cover_images.md)
- [`cover_images_reply`](cover_images_reply.md)
- [`db_value`](db_value.md)
- [`db_column`](db_column.md)
- [`db_row`](db_row.md)
- [`db_statement`](db_statement.md)
- [`execute_request`](execute_request.md)
- [`sql_span`](sql_span.md)
- [`text_collate_site`](text_collate_site.md)
- [`integer_arith_site`](integer_arith_site.md)
- [`physical_access`](physical_access.md)
- [`resolved_assignment`](resolved_assignment.md)
- [`named_sql_type`](named_sql_type.md)
- [`column_reference`](column_reference.md)
- [`optional_column_reference`](optional_column_reference.md)
- [`foreign_key_constraint`](foreign_key_constraint.md)
- [`table_constraint`](table_constraint.md)
- [`create_table_schema`](create_table_schema.md)
- [`schema_create`](schema_create.md)
- [`schema_action`](schema_action.md)
- [`resolved_statement`](resolved_statement.md)
- [`adapter_receipt`](adapter_receipt.md)
- [`adapter_statement`](adapter_statement.md)
- [`adapter_execute_request`](adapter_execute_request.md)
- [`statement_result`](statement_result.md)
- [`db_timing`](db_timing.md)
- [`execute_reply`](execute_reply.md)
- [`execute_result_reply`](execute_result_reply.md)
- [`db_capabilities`](db_capabilities.md)
- [`db_bootstrap_reply`](db_bootstrap_reply.md)
- [`db_bootstrap`](db_bootstrap.md)
- [`db_capabilities_reply`](db_capabilities_reply.md)
- [`database`](database/README.md)
- [`identity_high_water`](identity_high_water.md)
- [`identity_export_reply`](identity_export_reply.md)
- [`user_relations_reply`](user_relations_reply.md)
- [`adapter_database_session`](adapter_database_session/README.md)
- [`guest_database`](guest_database/README.md)
- [`plugin_migration_op`](plugin_migration_op.md)
- [`plugin_migration`](plugin_migration.md)
- [`plugin_migrations_ok`](plugin_migrations_ok.md)
- [`plugin_migrations_reply`](plugin_migrations_reply.md)
- [`plugin_worker`](plugin_worker/README.md)

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
    Metadata = 6,
    Covers = 7,
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

### WorkKind

```rust
pub enum WorkKind {
    Book = 0,
    PodcastShow = 1,
    PodcastEpisode = 2,
    Drama = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<WorkKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### ExpressionKind

```rust
pub enum ExpressionKind {
    Narration = 0,
    Text = 1,
    Translation = 2,
    Dramatisation = 3,
    PodcastAudio = 4,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<ExpressionKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### MediaKind

```rust
pub enum MediaKind {
    Audiobook = 0,
    Ebook = 1,
    Print = 2,
    PodcastAudio = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<MediaKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### IdentifierScheme

```rust
pub enum IdentifierScheme {
    Isbn = 0,
    Asin = 1,
    OpenLibraryEdition = 2,
    OpenLibraryWork = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<IdentifierScheme>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### CreditRole

```rust
pub enum CreditRole {
    Author = 0,
    Narrator = 1,
    Translator = 2,
    Editor = 3,
    Illustrator = 4,
    Publisher = 5,
    Host = 6,
    Cast = 7,
    Adaptor = 8,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<CreditRole>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### CreditLevel

```rust
pub enum CreditLevel {
    RoleDefault = 0,
    Work = 1,
    Expression = 2,
    Edition = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<CreditLevel>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

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

### MAX_CRON_BYTES

```rust
pub const MAX_CRON_BYTES: u32 = 128;
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

### MAX_COVER_URLS

```rust
pub const MAX_COVER_URLS: u32 = 32;
```

### MAX_COVER_BYTES

```rust
pub const MAX_COVER_BYTES: u32 = 10485760;
```

### MAX_COVER_HEADER_BYTES

```rust
pub const MAX_COVER_HEADER_BYTES: u32 = 1024;
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
