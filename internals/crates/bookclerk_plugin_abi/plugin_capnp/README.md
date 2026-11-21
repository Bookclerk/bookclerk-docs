# bookclerk_plugin_abi::plugin_capnp

Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).

Included at crate root because `capnpc` emits `crate::plugin_capnp` paths.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ObjectChangeKind`](#objectchangekind) | enum |  |
| [`SettingNetworkKind`](#settingnetworkkind) | enum |  |
| [`SettingGrantState`](#settinggrantstate) | enum |  |
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
| [`FetchedFileRole`](#fetchedfilerole) | enum |  |
| [`ChapterKind`](#chapterkind) | enum |  |
| [`CatalogSort`](#catalogsort) | enum |  |
| [`CatalogField`](#catalogfield) | enum |  |
| [`PriceLookup`](#pricelookup) | enum |  |
| [`Abridgement`](#abridgement) | enum |  |
| [`Explicitness`](#explicitness) | enum |  |
| [`OfferedCodec`](#offeredcodec) | enum |  |
| [`OfferedAacProfile`](#offeredaacprofile) | enum |  |
| [`MetadataField`](#metadatafield) | enum |  |
| [`NotificationSeverity`](#notificationseverity) | enum |  |
| [`NotificationAudience`](#notificationaudience) | enum |  |
| [`NotificationAlertAction`](#notificationalertaction) | enum |  |
| [`NotifyDestinationKind`](#notifydestinationkind) | enum |  |
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
| [`MAX_BATCH_KEYS`](#max_batch_keys) | constant |  |
| [`MAX_CHANGES_PAGE`](#max_changes_page) | constant |  |
| [`MAX_CHANGES_CURSOR_BYTES`](#max_changes_cursor_bytes) | constant |  |
| [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) | constant |  |
| [`MAX_SCAN_PAGE_BOOKS`](#max_scan_page_books) | constant |  |
| [`MAX_SCAN_CURSOR_BYTES`](#max_scan_cursor_bytes) | constant |  |
| [`MAX_REFRESHED_CREDENTIAL_BYTES`](#max_refreshed_credential_bytes) | constant |  |
| [`MAX_EVENT_CURSOR_BYTES`](#max_event_cursor_bytes) | constant |  |
| [`MAX_IDENTIFIER_BYTES`](#max_identifier_bytes) | constant |  |
| [`MAX_CRON_BYTES`](#max_cron_bytes) | constant |  |
| [`MAX_CONFIG_PAYLOAD_BYTES`](#max_config_payload_bytes) | constant |  |
| [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes) | constant |  |
| [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) | constant |  |
| [`MAX_COVER_URLS`](#max_cover_urls) | constant |  |
| [`MAX_COVER_BYTES`](#max_cover_bytes) | constant |  |
| [`MAX_COVER_HEADER_BYTES`](#max_cover_header_bytes) | constant |  |
| [`MAX_METADATA_FILE_BYTES`](#max_metadata_file_bytes) | constant |  |
| [`MAX_METADATA_FILE_DESCRIPTION_BYTES`](#max_metadata_file_description_bytes) | constant |  |
| [`MAX_METADATA_FILE_TEXT_BYTES`](#max_metadata_file_text_bytes) | constant |  |
| [`MAX_METADATA_FILE_LIST_ENTRIES`](#max_metadata_file_list_entries) | constant |  |
| [`MAX_METADATA_FILE_CHAPTERS`](#max_metadata_file_chapters) | constant |  |
| [`MAX_NOTIFICATION_TITLE_BYTES`](#max_notification_title_bytes) | constant |  |
| [`MAX_NOTIFICATION_TEXT_BYTES`](#max_notification_text_bytes) | constant |  |
| [`MAX_NOTIFICATION_HTML_BYTES`](#max_notification_html_bytes) | constant |  |
| [`MAX_NOTIFICATION_FIELDS`](#max_notification_fields) | constant |  |
| [`MAX_NOTIFICATION_FIELD_BYTES`](#max_notification_field_bytes) | constant |  |
| [`MAX_NOTIFIER_ADDRESS_BYTES`](#max_notifier_address_bytes) | constant |  |
| [`MAX_METADATA_AUDIO_FILES`](#max_metadata_audio_files) | constant |  |
| [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes) | constant |  |
| [`MAX_SOCKET_FRAME_BYTES`](#max_socket_frame_bytes) | constant |  |
| [`MAX_SOCKET_ATTACHMENT_BYTES`](#max_socket_attachment_bytes) | constant |  |
| [`MAX_SOCKETS_PER_INSTANCE`](#max_sockets_per_instance) | constant |  |
| [`MAX_SOCKET_AUTO_RESPONSES`](#max_socket_auto_responses) | constant |  |
| [`MAX_SOCKET_AUTO_RESPONSE_BYTES`](#max_socket_auto_response_bytes) | constant |  |
| [`MAX_SOCKET_URL_BYTES`](#max_socket_url_bytes) | constant |  |
| [`MAX_SOCKET_HEADERS`](#max_socket_headers) | constant |  |
| [`MAX_ORIGIN_HEADERS`](#max_origin_headers) | constant |  |
| [`MAX_ORIGIN_HEADER_BYTES`](#max_origin_header_bytes) | constant |  |
| [`MAX_ORIGIN_TARGET_BYTES`](#max_origin_target_bytes) | constant |  |
| [`MAX_INGRESS_ORIGIN_BYTES`](#max_ingress_origin_bytes) | constant |  |
| [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes) | constant |  |
| [`MAX_NETWORK_GRANT_ENTRIES`](#max_network_grant_entries) | constant |  |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant |  |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant |  |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant |  |
| [`FEATURE_STREAMS`](#feature_streams) | constant |  |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant |  |
| [`FEATURE_STORAGE_MOVE`](#feature_storage_move) | constant |  |
| [`FEATURE_STORAGE_MOVE_MANY`](#feature_storage_move_many) | constant |  |
| [`FEATURE_STORAGE_DELETE_MANY`](#feature_storage_delete_many) | constant |  |
| [`FEATURE_STORAGE_HEAD_MANY`](#feature_storage_head_many) | constant |  |
| [`FEATURE_STORAGE_CONDITIONAL_WRITE`](#feature_storage_conditional_write) | constant |  |
| [`FEATURE_STORAGE_CHANGES`](#feature_storage_changes) | constant |  |
| [`FEATURE_STORAGE_TRANSLATE_KEYS`](#feature_storage_translate_keys) | constant |  |

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
- [`move_options`](move_options.md)
- [`move_result`](move_result.md)
- [`key_pair`](key_pair.md)
- [`move_many_target`](move_many_target.md)
- [`move_many_request`](move_many_request.md)
- [`key_outcome`](key_outcome.md)
- [`move_many_page`](move_many_page.md)
- [`delete_many_result`](delete_many_result.md)
- [`head_many_result`](head_many_result.md)
- [`translate_keys_result`](translate_keys_result.md)
- [`changes_request`](changes_request.md)
- [`object_change`](object_change.md)
- [`changes_page`](changes_page.md)
- [`plugin_describe`](plugin_describe.md)
- [`optional`](optional.md)
- [`oidc_client_template`](oidc_client_template.md)
- [`oidc_clients_ok`](oidc_clients_ok.md)
- [`oidc_clients_reply`](oidc_clients_reply.md)
- [`extensible_config`](extensible_config.md)
- [`bindings`](bindings.md)
- [`network_grant`](network_grant.md)
- [`setting_network_grant`](setting_network_grant.md)
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
- [`event_read_request`](event_read_request.md)
- [`event_read_page`](event_read_page.md)
- [`event_read_reply`](event_read_reply.md)
- [`job_controller`](job_controller.md)
- [`head_ok`](head_ok.md)
- [`head_reply`](head_reply.md)
- [`list_reply`](list_reply.md)
- [`get_ok`](get_ok.md)
- [`get_reply`](get_reply.md)
- [`put_reply`](put_reply.md)
- [`copy_reply`](copy_reply.md)
- [`move_reply`](move_reply.md)
- [`move_many_reply`](move_many_reply.md)
- [`delete_many_reply`](delete_many_reply.md)
- [`head_many_reply`](head_many_reply.md)
- [`translate_keys_reply`](translate_keys_reply.md)
- [`changes_reply`](changes_reply.md)
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
- [`origin_response`](origin_response.md)
- [`origin_fetch_reply`](origin_fetch_reply.md)
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
- [`sockets`](sockets/README.md)
- [`socket_handler`](socket_handler/README.md)
- [`origin`](origin/README.md)
- [`ingress`](ingress/README.md)
- [`content_source`](content_source/README.md)
- [`remote_library`](remote_library/README.md)
- [`plugin_cli`](plugin_cli/README.md)
- [`oidc`](oidc/README.md)
- [`metadata_provider`](metadata_provider/README.md)
- [`cover_fetcher`](cover_fetcher/README.md)
- [`metadata_files`](metadata_files/README.md)
- [`notifier`](notifier/README.md)
- [`notification_raiser`](notification_raiser/README.md)
- [`json_enum`](json_enum.md)
- [`event_consumer_spec`](event_consumer_spec.md)
- [`plugin_capabilities`](plugin_capabilities.md)
- [`catalog_capabilities`](catalog_capabilities.md)
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
- [`media_support`](media_support.md)
- [`identifier`](identifier.md)
- [`credit`](credit.md)
- [`series_entry`](series_entry.md)
- [`scan_book`](scan_book.md)
- [`scan_page`](scan_page.md)
- [`scan_reply`](scan_reply.md)
- [`fetch_options`](fetch_options.md)
- [`fetch_title_params`](fetch_title_params.md)
- [`fetched_file`](fetched_file.md)
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
- [`priced_record`](priced_record.md)
- [`purchase_hint_reply`](purchase_hint_reply.md)
- [`purchase_hints_params`](purchase_hints_params.md)
- [`purchase_hints_result`](purchase_hints_result.md)
- [`purchase_hints_reply`](purchase_hints_reply.md)
- [`quality_offer`](quality_offer.md)
- [`quality_offers_params`](quality_offers_params.md)
- [`title_quality_offer`](title_quality_offer.md)
- [`quality_offers_result`](quality_offers_result.md)
- [`quality_offers_reply`](quality_offers_reply.md)
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
- [`metadata_file_record`](metadata_file_record.md)
- [`metadata_file_parse_params`](metadata_file_parse_params.md)
- [`metadata_file_parsed`](metadata_file_parsed.md)
- [`metadata_file_parse_reply`](metadata_file_parse_reply.md)
- [`metadata_audio_file`](metadata_audio_file.md)
- [`metadata_file_render_params`](metadata_file_render_params.md)
- [`metadata_file_rendered`](metadata_file_rendered.md)
- [`metadata_file_render_reply`](metadata_file_render_reply.md)
- [`notification_field`](notification_field.md)
- [`notification_message`](notification_message.md)
- [`notify_destination`](notify_destination.md)
- [`notify_send_params`](notify_send_params.md)
- [`notify_sent`](notify_sent.md)
- [`notify_send_reply`](notify_send_reply.md)
- [`notify_validate_params`](notify_validate_params.md)
- [`notify_validated`](notify_validated.md)
- [`notify_validate_reply`](notify_validate_reply.md)
- [`notify_raise_params`](notify_raise_params.md)
- [`notify_raised`](notify_raised.md)
- [`notify_raise_reply`](notify_raise_reply.md)
- [`socket_frame`](socket_frame.md)
- [`socket_header`](socket_header.md)
- [`socket_auto_response`](socket_auto_response.md)
- [`socket_connect_request`](socket_connect_request.md)
- [`socket_info`](socket_info.md)
- [`socket_connect_reply`](socket_connect_reply.md)
- [`socket_list`](socket_list.md)
- [`socket_list_reply`](socket_list_reply.md)
- [`socket_message_reply`](socket_message_reply.md)
- [`socket_message_ok`](socket_message_ok.md)
- [`socket_message`](socket_message.md)
- [`socket_closed`](socket_closed.md)
- [`origin_header`](origin_header.md)
- [`origin_request`](origin_request.md)
- [`ingress_started`](ingress_started.md)
- [`ingress_start_reply`](ingress_start_reply.md)
- [`ingress_health`](ingress_health.md)
- [`ingress_health_reply`](ingress_health_reply.md)
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
- [`execute_stream_head`](execute_stream_head.md)
- [`execute_stream_statement`](execute_stream_statement.md)
- [`execute_stream_frame`](execute_stream_frame.md)
- [`execute_stream_reply`](execute_stream_reply.md)
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

### ObjectChangeKind

```rust
pub enum ObjectChangeKind {
    Changed = 0,
    Upserted = 1,
    Removed = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<ObjectChangeKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### SettingNetworkKind

```rust
pub enum SettingNetworkKind {
    Fetch = 0,
    Tcp = 1,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<SettingNetworkKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### SettingGrantState

```rust
pub enum SettingGrantState {
    Approved = 0,
    Pending = 1,
    Unset = 2,
    Invalid = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<SettingGrantState>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

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
    LimitExceeded = 12,
    RateLimited = 13,
    NetworkNotGranted = 14,
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
    MetadataFiles = 8,
    Notifier = 9,
    Ingress = 10,
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
    Unspecified = 0,
    Book = 1,
    Podcast = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<WorkKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### ExpressionKind

```rust
pub enum ExpressionKind {
    Unspecified = 0,
    Text = 1,
    Narration = 2,
    Dramatisation = 3,
    Graphic = 4,
    OriginalAudio = 5,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<ExpressionKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### MediaKind

```rust
pub enum MediaKind {
    Unspecified = 0,
    Audio = 1,
    Ebook = 2,
    Print = 3,
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

### FetchedFileRole

```rust
pub enum FetchedFileRole {
    Unspecified = 0,
    Part = 1,
    Whole = 2,
    Cover = 3,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<FetchedFileRole>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### ChapterKind

```rust
pub enum ChapterKind {
    Unknown = 0,
    Chapters = 1,
    Tracks = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<ChapterKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

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

### PriceLookup

```rust
pub enum PriceLookup {
    Unspecified = 0,
    Product = 1,
    Isbn = 2,
    Asin = 3,
    Title = 4,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<PriceLookup>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### Abridgement

```rust
pub enum Abridgement {
    Unknown = 0,
    Unabridged = 1,
    Abridged = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<Abridgement>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### Explicitness

```rust
pub enum Explicitness {
    Unknown = 0,
    Clean = 1,
    Explicit = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<Explicitness>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### OfferedCodec

```rust
pub enum OfferedCodec {
    Unknown = 0,
    Aac = 1,
    Alac = 2,
    Flac = 3,
    Mp3 = 4,
    Opus = 5,
    Vorbis = 6,
    Eac3 = 7,
    Ac4 = 8,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<OfferedCodec>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### OfferedAacProfile

```rust
pub enum OfferedAacProfile {
    Unknown = 0,
    Lc = 1,
    He = 2,
    HeV2 = 3,
    Xhe = 4,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<OfferedAacProfile>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### MetadataField

```rust
pub enum MetadataField {
    Title = 0,
    Subtitle = 1,
    Description = 2,
    Publisher = 3,
    PublishedAt = 4,
    Language = 5,
    Genres = 6,
    Tags = 7,
    Abridgement = 8,
    ExpressionKind = 9,
    Identifiers = 10,
    Credits = 11,
    Series = 12,
    Chapters = 13,
    Duration = 14,
    Explicitness = 15,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<MetadataField>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### NotificationSeverity

```rust
pub enum NotificationSeverity {
    Info = 0,
    Warning = 1,
    Critical = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<NotificationSeverity>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### NotificationAudience

```rust
pub enum NotificationAudience {
    Operational = 0,
    Patron = 1,
    Account = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<NotificationAudience>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### NotificationAlertAction

```rust
pub enum NotificationAlertAction {
    None = 0,
    Trigger = 1,
    Resolve = 2,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<NotificationAlertAction>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

### NotifyDestinationKind

```rust
pub enum NotifyDestinationKind {
    Channel = 0,
    Personal = 1,
}
```

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `From<NotifyDestinationKind>`, `HasTypeId`, `Introspect`, `PartialEq`, `TryFrom<u16>`

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

### MAX_BATCH_KEYS

```rust
pub const MAX_BATCH_KEYS: u32 = 256;
```

### MAX_CHANGES_PAGE

```rust
pub const MAX_CHANGES_PAGE: u32 = 256;
```

### MAX_CHANGES_CURSOR_BYTES

```rust
pub const MAX_CHANGES_CURSOR_BYTES: u32 = 4096;
```

### MAX_CHECKPOINT_BYTES

```rust
pub const MAX_CHECKPOINT_BYTES: u32 = 65536;
```

### MAX_SCAN_PAGE_BOOKS

```rust
pub const MAX_SCAN_PAGE_BOOKS: u32 = 500;
```

### MAX_SCAN_CURSOR_BYTES

```rust
pub const MAX_SCAN_CURSOR_BYTES: u32 = 4096;
```

### MAX_REFRESHED_CREDENTIAL_BYTES

```rust
pub const MAX_REFRESHED_CREDENTIAL_BYTES: u32 = 65536;
```

### MAX_EVENT_CURSOR_BYTES

```rust
pub const MAX_EVENT_CURSOR_BYTES: u32 = 512;
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

### MAX_METADATA_FILE_BYTES

```rust
pub const MAX_METADATA_FILE_BYTES: u32 = 262144;
```

### MAX_METADATA_FILE_DESCRIPTION_BYTES

```rust
pub const MAX_METADATA_FILE_DESCRIPTION_BYTES: u32 = 65536;
```

### MAX_METADATA_FILE_TEXT_BYTES

```rust
pub const MAX_METADATA_FILE_TEXT_BYTES: u32 = 1024;
```

### MAX_METADATA_FILE_LIST_ENTRIES

```rust
pub const MAX_METADATA_FILE_LIST_ENTRIES: u32 = 256;
```

### MAX_METADATA_FILE_CHAPTERS

```rust
pub const MAX_METADATA_FILE_CHAPTERS: u32 = 2000;
```

### MAX_NOTIFICATION_TITLE_BYTES

```rust
pub const MAX_NOTIFICATION_TITLE_BYTES: u32 = 256;
```

### MAX_NOTIFICATION_TEXT_BYTES

```rust
pub const MAX_NOTIFICATION_TEXT_BYTES: u32 = 16384;
```

### MAX_NOTIFICATION_HTML_BYTES

```rust
pub const MAX_NOTIFICATION_HTML_BYTES: u32 = 65536;
```

### MAX_NOTIFICATION_FIELDS

```rust
pub const MAX_NOTIFICATION_FIELDS: u32 = 25;
```

### MAX_NOTIFICATION_FIELD_BYTES

```rust
pub const MAX_NOTIFICATION_FIELD_BYTES: u32 = 1024;
```

### MAX_NOTIFIER_ADDRESS_BYTES

```rust
pub const MAX_NOTIFIER_ADDRESS_BYTES: u32 = 4096;
```

### MAX_METADATA_AUDIO_FILES

```rust
pub const MAX_METADATA_AUDIO_FILES: u32 = 4096;
```

### MAX_KV_KEY_BYTES

```rust
pub const MAX_KV_KEY_BYTES: u32 = 512;
```

### MAX_SOCKET_FRAME_BYTES

```rust
pub const MAX_SOCKET_FRAME_BYTES: u32 = 262144;
```

### MAX_SOCKET_ATTACHMENT_BYTES

```rust
pub const MAX_SOCKET_ATTACHMENT_BYTES: u32 = 2048;
```

### MAX_SOCKETS_PER_INSTANCE

```rust
pub const MAX_SOCKETS_PER_INSTANCE: u32 = 8;
```

### MAX_SOCKET_AUTO_RESPONSES

```rust
pub const MAX_SOCKET_AUTO_RESPONSES: u32 = 4;
```

### MAX_SOCKET_AUTO_RESPONSE_BYTES

```rust
pub const MAX_SOCKET_AUTO_RESPONSE_BYTES: u32 = 256;
```

### MAX_SOCKET_URL_BYTES

```rust
pub const MAX_SOCKET_URL_BYTES: u32 = 2048;
```

### MAX_SOCKET_HEADERS

```rust
pub const MAX_SOCKET_HEADERS: u32 = 16;
```

### MAX_ORIGIN_HEADERS

```rust
pub const MAX_ORIGIN_HEADERS: u32 = 128;
```

### MAX_ORIGIN_HEADER_BYTES

```rust
pub const MAX_ORIGIN_HEADER_BYTES: u32 = 65536;
```

### MAX_ORIGIN_TARGET_BYTES

```rust
pub const MAX_ORIGIN_TARGET_BYTES: u32 = 8192;
```

### MAX_INGRESS_ORIGIN_BYTES

```rust
pub const MAX_INGRESS_ORIGIN_BYTES: u32 = 253;
```

### MAX_KV_METADATA_BYTES

```rust
pub const MAX_KV_METADATA_BYTES: u32 = 1024;
```

### MAX_NETWORK_GRANT_ENTRIES

```rust
pub const MAX_NETWORK_GRANT_ENTRIES: u32 = 256;
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

### FEATURE_STORAGE_MOVE

```rust
pub const FEATURE_STORAGE_MOVE: &str = "storage.move";
```

### FEATURE_STORAGE_MOVE_MANY

```rust
pub const FEATURE_STORAGE_MOVE_MANY: &str = "storage.moveMany";
```

### FEATURE_STORAGE_DELETE_MANY

```rust
pub const FEATURE_STORAGE_DELETE_MANY: &str = "storage.deleteMany";
```

### FEATURE_STORAGE_HEAD_MANY

```rust
pub const FEATURE_STORAGE_HEAD_MANY: &str = "storage.headMany";
```

### FEATURE_STORAGE_CONDITIONAL_WRITE

```rust
pub const FEATURE_STORAGE_CONDITIONAL_WRITE: &str = "storage.conditionalWrite";
```

### FEATURE_STORAGE_CHANGES

```rust
pub const FEATURE_STORAGE_CHANGES: &str = "storage.changes";
```

### FEATURE_STORAGE_TRANSLATE_KEYS

```rust
pub const FEATURE_STORAGE_TRANSLATE_KEYS: &str = "storage.translateKeys";
```
