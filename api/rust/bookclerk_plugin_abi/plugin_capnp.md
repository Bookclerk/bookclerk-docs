**bookclerk_plugin_abi > plugin_capnp**

# Module: plugin_capnp

## Contents

**Modules**

- [`account_credential`](#account_credential)
- [`adapter_database_session`](#adapter_database_session)
- [`adapter_execute_request`](#adapter_execute_request)
- [`adapter_receipt`](#adapter_receipt)
- [`adapter_session_reply`](#adapter_session_reply)
- [`adapter_statement`](#adapter_statement)
- [`authenticate_user_params`](#authenticate_user_params)
- [`bindings`](#bindings)
- [`brand`](#brand)
- [`byte_range`](#byte_range)
- [`byte_source`](#byte_source)
- [`cancellation`](#cancellation)
- [`cancelled_outcome`](#cancelled_outcome)
- [`catalog_detail`](#catalog_detail)
- [`catalog_detail_params`](#catalog_detail_params)
- [`catalog_detail_reply`](#catalog_detail_reply)
- [`catalog_hit`](#catalog_hit)
- [`catalog_hits`](#catalog_hits)
- [`catalog_hits_reply`](#catalog_hits_reply)
- [`chapter_marker`](#chapter_marker)
- [`cli_arg`](#cli_arg)
- [`cli_arg_spec`](#cli_arg_spec)
- [`cli_command_spec`](#cli_command_spec)
- [`cli_invoke_params`](#cli_invoke_params)
- [`cli_invoke_reply`](#cli_invoke_reply)
- [`cli_invoke_result`](#cli_invoke_result)
- [`cli_schema`](#cli_schema)
- [`cli_schema_reply`](#cli_schema_reply)
- [`column_reference`](#column_reference)
- [`completed_outcome`](#completed_outcome)
- [`config_option`](#config_option)
- [`config_option_value`](#config_option_value)
- [`content_source`](#content_source)
- [`copy_reply`](#copy_reply)
- [`copy_result`](#copy_result)
- [`create_table_schema`](#create_table_schema)
- [`database`](#database)
- [`database_adapter_config`](#database_adapter_config)
- [`db_bootstrap`](#db_bootstrap)
- [`db_bootstrap_reply`](#db_bootstrap_reply)
- [`db_capabilities`](#db_capabilities)
- [`db_capabilities_reply`](#db_capabilities_reply)
- [`db_column`](#db_column)
- [`db_row`](#db_row)
- [`db_statement`](#db_statement)
- [`db_timing`](#db_timing)
- [`db_value`](#db_value)
- [`describe_reply`](#describe_reply)
- [`destination`](#destination)
- [`diagnose_reply`](#diagnose_reply)
- [`diagnose_result`](#diagnose_result)
- [`domain_event`](#domain_event)
- [`empty_reply`](#empty_reply)
- [`entrypoints`](#entrypoints)
- [`entrypoints_reply`](#entrypoints_reply)
- [`event_ack`](#event_ack)
- [`event_batch`](#event_batch)
- [`event_batch_reply`](#event_batch_reply)
- [`event_consumer`](#event_consumer)
- [`event_consumer_spec`](#event_consumer_spec)
- [`event_dead_letter`](#event_dead_letter)
- [`event_poll_reply`](#event_poll_reply)
- [`event_poll_result`](#event_poll_result)
- [`event_publisher`](#event_publisher)
- [`event_reject`](#event_reject)
- [`event_result`](#event_result)
- [`event_retry`](#event_retry)
- [`event_suspended`](#event_suspended)
- [`execute_reply`](#execute_reply)
- [`execute_request`](#execute_request)
- [`execute_result_reply`](#execute_result_reply)
- [`expand_candidates_params`](#expand_candidates_params)
- [`extensible_config`](#extensible_config)
- [`external_user`](#external_user)
- [`external_user_reply`](#external_user_reply)
- [`fetch_options`](#fetch_options)
- [`fetch_title_params`](#fetch_title_params)
- [`fetch_title_reply`](#fetch_title_reply)
- [`foreign_key_constraint`](#foreign_key_constraint)
- [`get_ok`](#get_ok)
- [`get_reply`](#get_reply)
- [`guest_database`](#guest_database)
- [`guest_database_reply`](#guest_database_reply)
- [`handle_reply`](#handle_reply)
- [`head_ok`](#head_ok)
- [`head_reply`](#head_reply)
- [`health_ok`](#health_ok)
- [`health_reply`](#health_reply)
- [`identity_export_reply`](#identity_export_reply)
- [`identity_high_water`](#identity_high_water)
- [`integer_arith_site`](#integer_arith_site)
- [`invocation`](#invocation)
- [`job_controller`](#job_controller)
- [`job_invocation`](#job_invocation)
- [`job_outcome`](#job_outcome)
- [`job_runner`](#job_runner)
- [`json_enum`](#json_enum)
- [`kv_delete_ok`](#kv_delete_ok)
- [`kv_delete_reply`](#kv_delete_reply)
- [`kv_get_ok`](#kv_get_ok)
- [`kv_get_reply`](#kv_get_reply)
- [`kv_list_key`](#kv_list_key)
- [`kv_list_ok`](#kv_list_ok)
- [`kv_list_options`](#kv_list_options)
- [`kv_list_reply`](#kv_list_reply)
- [`kv_namespace`](#kv_namespace)
- [`kv_put_ok`](#kv_put_ok)
- [`kv_put_options`](#kv_put_options)
- [`kv_put_reply`](#kv_put_reply)
- [`list_deals_params`](#list_deals_params)
- [`list_options`](#list_options)
- [`list_page`](#list_page)
- [`list_reply`](#list_reply)
- [`listening_progress`](#listening_progress)
- [`login_complete_params`](#login_complete_params)
- [`login_params`](#login_params)
- [`login_reply`](#login_reply)
- [`login_result`](#login_result)
- [`login_start_reply`](#login_start_reply)
- [`login_start_result`](#login_start_result)
- [`named_database`](#named_database)
- [`named_kv`](#named_kv)
- [`named_queue`](#named_queue)
- [`named_sql_type`](#named_sql_type)
- [`object_info`](#object_info)
- [`object_metadata`](#object_metadata)
- [`oidc`](#oidc)
- [`oidc_client_template`](#oidc_client_template)
- [`oidc_clients_ok`](#oidc_clients_ok)
- [`oidc_clients_reply`](#oidc_clients_reply)
- [`open_ok`](#open_ok)
- [`open_reply`](#open_reply)
- [`optional`](#optional)
- [`optional_column_reference`](#optional_column_reference)
- [`physical_access`](#physical_access)
- [`plain_fetch`](#plain_fetch)
- [`plain_part`](#plain_part)
- [`plugin_capabilities`](#plugin_capabilities)
- [`plugin_cli`](#plugin_cli)
- [`plugin_describe`](#plugin_describe)
- [`plugin_error`](#plugin_error)
- [`plugin_event`](#plugin_event)
- [`plugin_migration`](#plugin_migration)
- [`plugin_migration_op`](#plugin_migration_op)
- [`plugin_migrations_ok`](#plugin_migrations_ok)
- [`plugin_migrations_reply`](#plugin_migrations_reply)
- [`plugin_worker`](#plugin_worker)
- [`progress_sink`](#progress_sink)
- [`publish_ok`](#publish_ok)
- [`publish_reply`](#publish_reply)
- [`pull_ok`](#pull_ok)
- [`pull_reply`](#pull_reply)
- [`purchase_hint`](#purchase_hint)
- [`purchase_hint_params`](#purchase_hint_params)
- [`purchase_hint_reply`](#purchase_hint_reply)
- [`purchase_hint_result`](#purchase_hint_result)
- [`put_reply`](#put_reply)
- [`put_result`](#put_result)
- [`queue_batch`](#queue_batch)
- [`queue_batch_ok`](#queue_batch_ok)
- [`queue_batch_reply`](#queue_batch_reply)
- [`queue_consumer`](#queue_consumer)
- [`queue_message`](#queue_message)
- [`queue_producer`](#queue_producer)
- [`queue_result`](#queue_result)
- [`queue_retry`](#queue_retry)
- [`queue_send_message`](#queue_send_message)
- [`queue_send_ok`](#queue_send_ok)
- [`queue_send_reply`](#queue_send_reply)
- [`read_options`](#read_options)
- [`rejected_outcome`](#rejected_outcome)
- [`remote_library`](#remote_library)
- [`resolved_assignment`](#resolved_assignment)
- [`resolved_statement`](#resolved_statement)
- [`retryable_outcome`](#retryable_outcome)
- [`scalar_limits`](#scalar_limits)
- [`scan_book`](#scan_book)
- [`scan_library_params`](#scan_library_params)
- [`scan_params`](#scan_params)
- [`scan_reply`](#scan_reply)
- [`scan_summary`](#scan_summary)
- [`schema_action`](#schema_action)
- [`schema_create`](#schema_create)
- [`search_catalog_params`](#search_catalog_params)
- [`source`](#source)
- [`source_account`](#source_account)
- [`source_accounts`](#source_accounts)
- [`source_accounts_reply`](#source_accounts_reply)
- [`sql_span`](#sql_span)
- [`statement_result`](#statement_result)
- [`suspended_outcome`](#suspended_outcome)
- [`sync_listening_reply`](#sync_listening_reply)
- [`sync_listening_result`](#sync_listening_result)
- [`table_constraint`](#table_constraint)
- [`text_collate_site`](#text_collate_site)
- [`user_relations_reply`](#user_relations_reply)
- [`write_options`](#write_options)

**Enums**

- [`Abridgement`](#abridgement)
- [`CatalogField`](#catalogfield)
- [`CatalogSort`](#catalogsort)
- [`CliArgKind`](#cliargkind)
- [`DbResultSelection`](#dbresultselection)
- [`DbStatementKind`](#dbstatementkind)
- [`DbType`](#dbtype)
- [`Entrypoint`](#entrypoint)
- [`IntegerArithKind`](#integerarithkind)
- [`IsolationReq`](#isolationreq)
- [`PluginErrorCode`](#pluginerrorcode)
- [`PortalAuthMode`](#portalauthmode)
- [`ResolvedSqlType`](#resolvedsqltype)

**Constants**

- [`API_VERSION`](#api_version)
- [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits)
- [`FEATURE_STORAGE_COPY`](#feature_storage_copy)
- [`FEATURE_STREAMS`](#feature_streams)
- [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes)
- [`MAX_CONFIG_PAYLOAD_BYTES`](#max_config_payload_bytes)
- [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes)
- [`MAX_IDENTIFIER_BYTES`](#max_identifier_bytes)
- [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes)
- [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes)
- [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes)
- [`MAX_LIST_PAGE`](#max_list_page)
- [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops)
- [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes)
- [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops)
- [`MAX_SCALAR_BYTES`](#max_scalar_bytes)
- [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes)

---

## bookclerk_plugin_abi::plugin_capnp::API_VERSION

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::Abridgement

*Enum*

**Variants:**
- `Unknown`
- `Unabridged`
- `Abridged`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Abridgement`
- **PartialEq**
  - `fn eq(self: &Self, other: &Abridgement) -> bool`
- **From**
  - `fn from(value: Abridgement) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <Abridgement as ::core::convert::TryFrom>::Error>`



## bookclerk_plugin_abi::plugin_capnp::CatalogField

*Enum*

**Variants:**
- `Any`
- `Author`
- `Narrator`
- `Series`
- `Genre`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **From**
  - `fn from(value: CatalogField) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <CatalogField as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogField`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogField) -> bool`



## bookclerk_plugin_abi::plugin_capnp::CatalogSort

*Enum*

**Variants:**
- `Relevance`
- `Popularity`
- `Rating`
- `Title`
- `Author`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <CatalogSort as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSort`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSort) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **From**
  - `fn from(value: CatalogSort) -> Self`



## bookclerk_plugin_abi::plugin_capnp::CliArgKind

*Enum*

**Variants:**
- `String`
- `Bool`
- `Int`
- `Path`

**Traits:** HasTypeId, Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CliArgKind`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliArgKind) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <CliArgKind as ::core::convert::TryFrom>::Error>`
- **From**
  - `fn from(value: CliArgKind) -> Self`



## bookclerk_plugin_abi::plugin_capnp::DbResultSelection

*Enum*

**Variants:**
- `Discard`
- `AffectedRows`
- `Rows`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <DbResultSelection as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DbResultSelection`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbResultSelection) -> bool`



## bookclerk_plugin_abi::plugin_capnp::DbStatementKind

*Enum*

**Variants:**
- `Execute`
- `Select`
- `Returning`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <DbStatementKind as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DbStatementKind`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbStatementKind) -> bool`



## bookclerk_plugin_abi::plugin_capnp::DbType

*Enum*

**Variants:**
- `Unspecified`
- `Bool`
- `Int64`
- `Float64`
- `Text`
- `Bytes`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <DbType as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DbType`
- **PartialEq**
  - `fn eq(self: &Self, other: &DbType) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`



## bookclerk_plugin_abi::plugin_capnp::Entrypoint

*Enum*

**Variants:**
- `Storefront`
- `Storage`
- `DatabaseAdapter`
- `RemoteLibrary`
- `Cli`
- `Oidc`

**Traits:** HasTypeId, Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Entrypoint`
- **PartialEq**
  - `fn eq(self: &Self, other: &Entrypoint) -> bool`
- **From**
  - `fn from(value: Entrypoint) -> Self`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <Entrypoint as ::core::convert::TryFrom>::Error>`



## bookclerk_plugin_abi::plugin_capnp::FEATURE_SCALAR_LIMITS

*Constant*: `&str`



## bookclerk_plugin_abi::plugin_capnp::FEATURE_STORAGE_COPY

*Constant*: `&str`



## bookclerk_plugin_abi::plugin_capnp::FEATURE_STREAMS

*Constant*: `&str`



## bookclerk_plugin_abi::plugin_capnp::IntegerArithKind

*Enum*

**Variants:**
- `Add`
- `Sub`
- `Mul`
- `Abs`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <IntegerArithKind as ::core::convert::TryFrom>::Error>`
- **Clone**
  - `fn clone(self: &Self) -> IntegerArithKind`
- **PartialEq**
  - `fn eq(self: &Self, other: &IntegerArithKind) -> bool`



## bookclerk_plugin_abi::plugin_capnp::IsolationReq

*Enum*

**Variants:**
- `AtomicBatch`
- `NestedSavepoint`
- `ConsistentSnapshot`

**Traits:** HasTypeId, Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> IsolationReq`
- **PartialEq**
  - `fn eq(self: &Self, other: &IsolationReq) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <IsolationReq as ::core::convert::TryFrom>::Error>`



## bookclerk_plugin_abi::plugin_capnp::MAX_CHECKPOINT_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_CONFIG_PAYLOAD_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_EVENT_PAYLOAD_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_IDENTIFIER_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_KV_KEY_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_KV_METADATA_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_KV_VALUE_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_LIST_PAGE

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_PLUGIN_MIGRATION_OPS

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_PLUGIN_MIGRATION_TOTAL_OPS

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_SCALAR_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::MAX_STREAM_WINDOW_BYTES

*Constant*: `u32`



## bookclerk_plugin_abi::plugin_capnp::PluginErrorCode

*Enum*

**Variants:**
- `InvalidParams`
- `Unauthorized`
- `Forbidden`
- `NotFound`
- `Unavailable`
- `Unsupported`
- `Internal`
- `PayloadTooLarge`
- `DeadlineExceeded`
- `InvalidCursor`
- `Cancelled`
- `Conflict`

**Traits:** HasTypeId, Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PluginErrorCode`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginErrorCode) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <PluginErrorCode as ::core::convert::TryFrom>::Error>`



## bookclerk_plugin_abi::plugin_capnp::PortalAuthMode

*Enum*

**Variants:**
- `Unspecified`
- `Password`
- `Oauth`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <PortalAuthMode as ::core::convert::TryFrom>::Error>`
- **From**
  - `fn from(value: PortalAuthMode) -> Self`
- **Clone**
  - `fn clone(self: &Self) -> PortalAuthMode`
- **PartialEq**
  - `fn eq(self: &Self, other: &PortalAuthMode) -> bool`



## bookclerk_plugin_abi::plugin_capnp::ResolvedSqlType

*Enum*

**Variants:**
- `Integer`
- `Real`
- `Text`
- `Blob`
- `Boolean`
- `Null`

**Traits:** Copy, Eq, HasTypeId

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ResolvedSqlType`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolvedSqlType) -> bool`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Introspect**
  - `fn introspect() -> ::capnp::introspect::Type`
- **TryFrom**
  - `fn try_from(value: u16) -> ::core::result::Result<Self, <ResolvedSqlType as ::core::convert::TryFrom>::Error>`



## Module: account_credential



## Module: adapter_database_session



## Module: adapter_execute_request



## Module: adapter_receipt



## Module: adapter_session_reply



## Module: adapter_statement



## Module: authenticate_user_params



## Module: bindings



## Module: brand



## Module: byte_range



## Module: byte_source



## Module: cancellation



## Module: cancelled_outcome



## Module: catalog_detail



## Module: catalog_detail_params



## Module: catalog_detail_reply



## Module: catalog_hit



## Module: catalog_hits



## Module: catalog_hits_reply



## Module: chapter_marker



## Module: cli_arg



## Module: cli_arg_spec



## Module: cli_command_spec



## Module: cli_invoke_params



## Module: cli_invoke_reply



## Module: cli_invoke_result



## Module: cli_schema



## Module: cli_schema_reply



## Module: column_reference



## Module: completed_outcome



## Module: config_option



## Module: config_option_value



## Module: content_source



## Module: copy_reply



## Module: copy_result



## Module: create_table_schema



## Module: database



## Module: database_adapter_config



## Module: db_bootstrap



## Module: db_bootstrap_reply



## Module: db_capabilities



## Module: db_capabilities_reply



## Module: db_column



## Module: db_row



## Module: db_statement



## Module: db_timing



## Module: db_value



## Module: describe_reply



## Module: destination



## Module: diagnose_reply



## Module: diagnose_result



## Module: domain_event



## Module: empty_reply



## Module: entrypoints



## Module: entrypoints_reply



## Module: event_ack



## Module: event_batch



## Module: event_batch_reply



## Module: event_consumer



## Module: event_consumer_spec



## Module: event_dead_letter



## Module: event_poll_reply



## Module: event_poll_result



## Module: event_publisher



## Module: event_reject



## Module: event_result



## Module: event_retry



## Module: event_suspended



## Module: execute_reply



## Module: execute_request



## Module: execute_result_reply



## Module: expand_candidates_params



## Module: extensible_config



## Module: external_user



## Module: external_user_reply



## Module: fetch_options



## Module: fetch_title_params



## Module: fetch_title_reply



## Module: foreign_key_constraint



## Module: get_ok



## Module: get_reply



## Module: guest_database



## Module: guest_database_reply



## Module: handle_reply



## Module: head_ok



## Module: head_reply



## Module: health_ok



## Module: health_reply



## Module: identity_export_reply



## Module: identity_high_water



## Module: integer_arith_site



## Module: invocation



## Module: job_controller



## Module: job_invocation



## Module: job_outcome



## Module: job_runner



## Module: json_enum



## Module: kv_delete_ok



## Module: kv_delete_reply



## Module: kv_get_ok



## Module: kv_get_reply



## Module: kv_list_key



## Module: kv_list_ok



## Module: kv_list_options



## Module: kv_list_reply



## Module: kv_namespace



## Module: kv_put_ok



## Module: kv_put_options



## Module: kv_put_reply



## Module: list_deals_params



## Module: list_options



## Module: list_page



## Module: list_reply



## Module: listening_progress



## Module: login_complete_params



## Module: login_params



## Module: login_reply



## Module: login_result



## Module: login_start_reply



## Module: login_start_result



## Module: named_database



## Module: named_kv



## Module: named_queue



## Module: named_sql_type



## Module: object_info



## Module: object_metadata



## Module: oidc



## Module: oidc_client_template



## Module: oidc_clients_ok



## Module: oidc_clients_reply



## Module: open_ok



## Module: open_reply



## Module: optional



## Module: optional_column_reference



## Module: physical_access



## Module: plain_fetch



## Module: plain_part



## Module: plugin_capabilities



## Module: plugin_cli



## Module: plugin_describe



## Module: plugin_error



## Module: plugin_event



## Module: plugin_migration



## Module: plugin_migration_op



## Module: plugin_migrations_ok



## Module: plugin_migrations_reply



## Module: plugin_worker



## Module: progress_sink



## Module: publish_ok



## Module: publish_reply



## Module: pull_ok



## Module: pull_reply



## Module: purchase_hint



## Module: purchase_hint_params



## Module: purchase_hint_reply



## Module: purchase_hint_result



## Module: put_reply



## Module: put_result



## Module: queue_batch



## Module: queue_batch_ok



## Module: queue_batch_reply



## Module: queue_consumer



## Module: queue_message



## Module: queue_producer



## Module: queue_result



## Module: queue_retry



## Module: queue_send_message



## Module: queue_send_ok



## Module: queue_send_reply



## Module: read_options



## Module: rejected_outcome



## Module: remote_library



## Module: resolved_assignment



## Module: resolved_statement



## Module: retryable_outcome



## Module: scalar_limits



## Module: scan_book



## Module: scan_library_params



## Module: scan_params



## Module: scan_reply



## Module: scan_summary



## Module: schema_action



## Module: schema_create



## Module: search_catalog_params



## Module: source



## Module: source_account



## Module: source_accounts



## Module: source_accounts_reply



## Module: sql_span



## Module: statement_result



## Module: suspended_outcome



## Module: sync_listening_reply



## Module: sync_listening_result



## Module: table_constraint



## Module: text_collate_site



## Module: user_relations_reply



## Module: write_options



