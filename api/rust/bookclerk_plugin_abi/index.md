# bookclerk_plugin_abi

Authoritative Bookclerk plugin ABI (`api_version` 3).

Version 3 is the capability-driven object-capability ABI (Cap'n Proto RPC,
transferred byte streams). The guest root is [`PluginWorker`]:
`describe()` advertises typed [`PluginCapabilities`](generated.md#plugincapabilities), then
`open(invocation, bindings)` returns the exported [`Entrypoints`](#entrypoints)
(`storefront`, `storage`, `databaseAdapter`, `remoteLibrary`, `cli`,
`oidc`, plus the `eventConsumer` / `jobRunner` triggers). Host-granted
[`Bindings`](#bindings) carry `CONFIG` / `SECRETS`, the `EVENTS` publisher, and named
`[[databases]]` sessions — the Workers `env` idiom.

## Audience

- **Guest authors** — implement [`PluginWorker`](#pluginworker) and the entrypoint traits
  against these DTOs (via `bookclerk-plugin-sdk`, `@bookclerk/plugin-sdk`,
  or a language binding generated from the same schema).
- **Host / SDK maintainers** — drive entrypoint capabilities, seal
  credentials, and upsert library rows without depending on store-specific
  crates.

Product narrative (jail, consent, install layout) lives in
[`docs/plugins/README.md`](https://github.com/bookclerk/bookclerk/blob/main/docs/plugins/README.md).
This crate is the typed wire contract only.

## Schema

The Cap'n Proto schema at
[`schema/plugin.capnp`](https://github.com/bookclerk/bookclerk/blob/main/crates/bookclerk-plugin-abi/schema/plugin.capnp)
is the single source of truth: RPC interfaces, product constants, database
enums, and the "Typed method payloads" section. Types here are the Rust
projection; the TypeScript and
Python SDK projections are generated from the same schema by
`scripts/gen-plugin-abi.py`, which also drift-checks this crate. Wire DTO
fields serialize as **camelCase**.

Install manifests validate against [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json)
(`schema/plugin-toml.json`).

## Versioning

[`PRODUCT_API_VERSION`](#product_api_version) is `3` (capability-driven Cap'n Proto / Workers
RPC). Product spawn requires `plugin.toml` `api_version = 3`. There is no
`protocol` key.

## Modules

| Module | Contents |
| --- | --- |
| [`methods`](methods.md) | Capability / consent method name constants (`login`, `event`, …) |
| Crate-root DTOs | [`PluginDescribe`](#plugindescribe), health, CLI, destination objects |
| [`kind`](kind.md) | Kind-specific DTOs (source / integration / output) |
| [`settings_schema`](settings_schema.md) | `[[settings]]` operator settings schema shared by manifest and host |
| [`package_version`](package_version.md) | `plugin.toml` `version` parsing and ordering for schema pinning |
| [`db`](db.md) | Host-private database connect params (feature `host`) |
| [`error`](error.md) | [`PluginError`](error.md#pluginerror) / [`PluginErrorCode`](error.md#pluginerrorcode) |
| [`guest_sql`](guest_sql.md) | SQL-v1 grammar / guest admission |
| [`sql_desugar`](sql_desugar.md) | Host-only semantic desugars (`NULLS`, `NULLIF`) |
| [`plugin_capnp`](plugin_capnp.md) | Generated Cap'n Proto RPC interfaces |

| Item | Kind | Summary |
| --- | --- | --- |
| [`AdapterTransaction`](#adaptertransaction) | trait | Host-only interactive transaction on an adapter connection (SeaORM proxy). |
| [`HostAdapterDatabaseSession`](#hostadapterdatabasesession) | trait | Host-only view of an open adapter session (`begin` for interactive txn). |
| [`AdapterDatabaseSession`](#adapterdatabasesession) | trait | Host ↔ database adapter session (`capabilities` + typed `execute`). |
| [`Cancellation`](#cancellation) | trait | Transport cancellation. SDKs project this into a locally created `AbortSignal` (AbortSignal is not a serializable Workers RPC value). |
| [`ContentSource`](#contentsource) | trait | Storefront content source (not byte `Source`). |
| [`Database`](#database) | trait | Database factory. Sessions cannot survive suspension. |
| [`Destination`](#destination) | trait | Destination capability (storage). |
| [`EventConsumer`](#eventconsumer) | trait | `[[events.consumers]]` handler (`Entrypoints::event_consumer`). |
| [`EventPublisher`](#eventpublisher) | trait | `EVENTS` binding: host-served outbox publisher (`Bindings::events`). |
| [`GuestDatabase`](#guestdatabase) | trait | Host-granted SQL transport for job plugin authors (no `capabilities`). |
| [`JobRunner`](#jobrunner) | trait | `[triggers] jobs` handler (`Entrypoints::job_runner`). |
| [`KvNamespace`](#kvnamespace) | trait | Durable key/value namespace: Workers `KVNamespace`-shaped binding the host serves from the library database, scoped to the plugin and the namespace a `[[kv_namespaces]]` row names. Values live in the cluster, so an instance moved to another host reads its own keys. |
| [`Oidc`](#oidc) | trait | `oidc` entrypoint: relying-party client templates and credential verification on behalf of the host authorization server. |
| [`PluginCli`](#plugincli) | trait | `cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`. |
| [`PluginWorker`](#pluginworker) | trait | Root `PluginWorker` capability (`describe` / `open` / `shutdown`). |
| [`ProgressSink`](#progresssink) | trait | Progress reports for a job invocation (never carries media). |
| [`QueueConsumer`](#queueconsumer) | trait | `[[queues.consumers]]` handler (`Entrypoints::queue_consumer`). |
| [`QueueProducer`](#queueproducer) | trait | Queue producer binding: host-served send into the queue a `[[queues.producers]]` row names (`Bindings::queues`). |
| [`RemoteLibrary`](#remotelibrary) | trait | `remoteLibrary` entrypoint: long-running remote-library lifecycle. |
| [`Source`](#source) | trait | Source capability that can open a named object as a stream. |
| [`GuestReceiptPersist`](#guestreceiptpersist) | struct | Host-stamped hint for adapters to persist guest replay payload before COMMIT. |
| [`AdapterExecuteRequest`](#adapterexecuterequest) | struct | Adapter execute payload: canonical SQL, structured proofs, optional receipt. |
| [`CanonicalExecuteRequest`](#canonicalexecuterequest) | struct | Host-desugared canonical SQL. Proofs are not yet bound. |
| [`UnresolvedExecuteRequest`](#unresolvedexecuterequest) | struct | Source SQL that has not been host-canonicalized. |
| [`PluginMigration`](#pluginmigration) | struct | One plugin-owned migration application in registration order. |
| [`IntegerArithSite`](#integerarithsite) | struct | One INTEGER arithmetic expression that must not wrap or error on overflow. |
| [`PhysicalAccess`](#physicalaccess) | struct | Physical table/column access used for authorization. |
| [`ResolvedAssignment`](#resolvedassignment) | struct | Destination assignment `lhs = rhs` (INSERT, UPDATE, …). |
| [`ResolvedStatement`](#resolvedstatement) | struct | One resolved typed proof bound to an exact canonical statement. |
| [`SqlSpan`](#sqlspan) | struct | Byte span in the exact canonical SQL string a proof is bound to. |
| [`TextCollateSite`](#textcollatesite) | struct | One TEXT expression that PostgreSQL must collate as `"C"`. |
| [`SqlFnCall`](#sqlfncall) | struct | One helper call found in canonical SQL (code spans only). |
| [`HostAdapterDatabaseSessionClient`](#hostadapterdatabasesessionclient) | struct | Host-side client for `begin` on an open adapter session capability. |
| [`RpcFeature`](#rpcfeature) | struct | Known RPC feature names. |
| [`StreamCopyHandler`](#streamcopyhandler) | struct | Default `JobRunner` for the stream-copy vertical slice. |
| [`StreamCopySpec`](#streamcopyspec) | struct | JSON body for a `stream_copy` `JobInvocation`. |
| [`ScalarLimits`](#scalarlimits) | struct | Negotiated numeric limits advertised at `super::PluginDescribe`. |
| [`BindingValues`](#bindingvalues) | struct | Plain-data portion of `Bindings`: every granted value that is not a capability. Hosts build this off the vat thread and attach capabilities when they call `open`. |
| [`Bindings`](#bindings) | struct | Host-granted bindings for one `PluginWorker::open` (`env` in the Workers idiom). Capability fields are `None` when the manifest does not declare (or the operator did not grant) the binding. |
| [`ByteRange`](#byterange) | struct | Inclusive byte range for a streamed read. |
| [`Entrypoints`](#entrypoints) | struct | Exported entrypoints returned by `PluginWorker::open`: one capability per `plugin.toml` `entrypoints` entry / trigger, `None` when not exported. |
| [`JobController`](#jobcontroller) | struct | Everything one `JobRunner::job` invocation may touch. |
| [`NamedKvBinding`](#namedkvbinding) | struct | One granted `[[kv_namespaces]]` binding on `Bindings::kv`. |
| [`NamedQueueBinding`](#namedqueuebinding) | struct | One granted `[[queues.producers]]` binding on `Bindings::queues`. |
| [`NeverCancel`](#nevercancel) | struct | Cancellation that never fires (tests / hosts without a fence). |
| [`ReadResult`](#readresult) | struct | Streamed read result. `body` ownership is transferred to the caller. |
| [`AdapterSessionHandle`](#adaptersessionhandle) | struct | Public adapter session plus host-private interactive transaction client. |
| [`ContentSourceClient`](#contentsourceclient) | struct | Cap'n Proto client for `ContentSource`. |
| [`DatabaseClient`](#databaseclient) | struct | Cap'n Proto client for `Database`. |
| [`DestinationClient`](#destinationclient) | struct | Host-side `Destination` over a capnp destination stub. |
| [`DestinationServer`](#destinationserver) | struct | Cap'n Proto server wrapping a `Destination` trait object. |
| [`EventConsumerClient`](#eventconsumerclient) | struct | Cap'n Proto client for `EventConsumer` (host → guest delivery). |
| [`EventPublisherClient`](#eventpublisherclient) | struct | Guest-side client for the host `EVENTS` binding (`EventPublisher`). |
| [`HostBindings`](#hostbindings) | struct | Host-side `Bindings`: plain values plus host-served capabilities the guest receives on `open`. |
| [`JobRunnerClient`](#jobrunnerclient) | struct | Cap'n Proto client for `JobRunner` (host → guest job dispatch). |
| [`KvNamespaceClient`](#kvnamespaceclient) | struct | Guest-side client for a host KV namespace binding (`KvNamespace`). |
| [`OidcClient`](#oidcclient) | struct | Cap'n Proto client for `Oidc`. |
| [`OpenedEntrypoints`](#openedentrypoints) | struct | Typed clients for the entrypoints a guest exported from `open`. `None` means the guest did not export that entrypoint. |
| [`PluginCliClient`](#plugincliclient) | struct | Cap'n Proto client for `PluginCli`. |
| [`PluginClient`](#pluginclient) | struct | Host bootstrap client for a plugin vat. |
| [`PluginServer`](#pluginserver) | struct | Cap'n Proto server wrapping `PluginWorker`. |
| [`QueueConsumerClient`](#queueconsumerclient) | struct | Cap'n Proto client for `QueueConsumer` (host → guest delivery). |
| [`QueueProducerClient`](#queueproducerclient) | struct | Guest-side client for a host queue producer binding (`QueueProducer`). |
| [`RemoteLibraryClient`](#remotelibraryclient) | struct | Cap'n Proto client for `RemoteLibrary`. |
| [`SourceClient`](#sourceclient) | struct | Host-side `Source` over a capnp source stub. |
| [`SourceServer`](#sourceserver) | struct | Cap'n Proto server wrapping a `Source`. |
| [`CopyResult`](#copyresult) | struct | Result of a server-side copy. |
| [`DomainEvent`](#domainevent) | struct | Domain event (not a job). Outbox-produced, at-least-once, idempotent consume. |
| [`ExtensibleConfig`](#extensibleconfig) | struct | Versioned plugin-specific config (not a substitute for typed ABI fields). |
| [`HealthOk`](#healthok) | struct | Health payload. |
| [`JobCheckpoint`](#jobcheckpoint) | struct | Bounded, versioned checkpoint attached to an invocation or suspension. |
| [`JobInvocation`](#jobinvocation) | struct | Versioned durable command envelope (not a domain event). |
| [`JobInvocationLease`](#jobinvocationlease) | struct | Claimed-lease fields used to populate a `JobInvocation`. |
| [`ListOptions`](#listoptions) | struct | Paginated list request. |
| [`ListPage`](#listpage) | struct | One page of keys. |
| [`ObjectInfo`](#objectinfo) | struct | Object listing entry. |
| [`ObjectMetadata`](#objectmetadata) | struct | Metadata without a body. |
| [`OidcClientTemplate`](#oidcclienttemplate) | struct | Plugin-declared Bookclerk-as-IdP relying-party template. |
| [`PluginDescribe`](#plugindescribe) | struct | Guest identity returned by `PluginWorker.describe` (the adapter merges the manifest projection over the author `describe()`). |
| [`PutResult`](#putresult) | struct | Result of a streamed put. |
| [`QueryPage`](#querypage) | struct | Bounded query page for host-internal SeaORM session helpers (not Cap'n ABI). |
| [`ScalarLimitsDto`](#scalarlimitsdto) | struct | Wire form of `ScalarLimits`. |
| [`WriteOptions`](#writeoptions) | struct | Write options for `Destination.put`. |
| [`PluginMigrationOp`](#pluginmigrationop) | enum | One already-separated BookclerkSQL operation in a registered migration. |
| [`OverflowDialect`](#overflowdialect) | enum | Engine family for overflow derived-table syntax and i64::MIN CAST. |
| [`IntegerArithKind`](#integerarithkind) | enum | INTEGER `+` `-` `*` `abs` site lowered to overflow → NULL. |
| [`SchemaAction`](#schemaaction) | enum | CREATE/DROP action recorded on a proof. |
| [`LikePatternSrc`](#likepatternsrc) | enum | Proven LIKE pattern sources in one canonical statement. |
| [`EventResult`](#eventresult) | enum | Per-event result of `crate::EventConsumer::event`. |
| [`JobOutcome`](#joboutcome) | enum | Outcome of `crate::JobRunner::job`. |
| [`capnp_u32_len`](#capnp_u32_len) | function | Converts a Rust list length into a Cap'n Proto `UInt32` list size. |
| [`plugin_migration_registration_bytes`](#plugin_migration_registration_bytes) | function | Aggregate UTF-8 bytes of ids plus SQL across a complete registration. |
| [`require_plugin_migration_json_lists`](#require_plugin_migration_json_lists) | function | Rejects JSON `databaseMigrations` list sizes before deserializing operations. |
| [`require_plugin_migration_registration`](#require_plugin_migration_registration) | function | Rejects a plugin migration registration that exceeds RPC resource limits. |
| [`apply_integer_overflow`](#apply_integer_overflow) | function | Wrap each INTEGER `+`/`-`/`*`/`abs` site in a portable overflow CASE. |
| [`assert_proof_matches_sql`](#assert_proof_matches_sql) | function | Require `proof` to be bound to `sql`. |
| [`admitted_bookclerk_sql_samples`](#admitted_bookclerk_sql_samples) | function | Grammar-aware BookclerkSQL samples for fuzz corpora and differential tests. |
| [`canonical_statements_checksum`](#canonical_statements_checksum) | function | Length-prefixed SHA-256 of an ordered canonical statement list. |
| [`glob_expanded_like_pattern_bytes`](#glob_expanded_like_pattern_bytes) | function | UTF-8 byte length of a sqlite-family GLOB pattern after Bookclerk LIKE→GLOB metacharacter escaping (`[` `*` `?` become three bytes). |
| [`like_pattern_sources`](#like_pattern_sources) | function | Collects LIKE / NOT LIKE pattern sources with proven lengths when possible. |
| [`require_function_args_within`](#require_function_args_within) | function | Enforces `crate::DbCapabilities::max_function_args` on non-chunkable helpers. |
| [`require_like_patterns_within`](#require_like_patterns_within) | function | Enforces `crate::DbCapabilities::max_pattern_bytes` on LIKE patterns. |
| [`require_portable_text`](#require_portable_text) | function | Rejects BookclerkSQL `TEXT` that contains U+0000. |
| [`require_portable_text_binds`](#require_portable_text_binds) | function | Rejects U+0000 in every BookclerkSQL `TEXT` bind. |
| [`sql_v1_function_calls`](#sql_v1_function_calls) | function | Collects helper calls in code spans (strings/comments are skipped). |
| [`sql_v1_helper_is_chunkable`](#sql_v1_helper_is_chunkable) | function | Helpers adapters may nest so each physical call stays inside `crate::DbCapabilities::max_function_args`. |
| [`sql_v1_pack_statements`](#sql_v1_pack_statements) | function | Splits a BookclerkSQL pack into statements using the SQL-v1 lexer. |
| [`sqlite_family_like_divmod_insert_upper_bound`](#sqlite_family_like_divmod_insert_upper_bound) | function | Upper bound on sqlite-family mechanical lowering length for a canonical statement of `canonical_len` bytes (`div`/`mod` NULLIF, `INSERT OR IGNORE`, LIKE→GLOB). Does not include proof-directed INTEGER overflow wraps. |
| [`sqlite_family_mechanical_len_upper_bound`](#sqlite_family_mechanical_len_upper_bound) | function | Mechanical sqlite-family length bound for one canonical statement, counting actual `LIKE` occurrences, `/` `%` operators in code spans, and at most one `INSERT OR IGNORE` rewrite. |
| [`text_contains_nul`](#text_contains_nul) | function | True when `text` contains U+0000. |
| [`canonical_execute_request_hash`](#canonical_execute_request_hash) | function | SHA-256 hex of the Cap'n-encoded request with transport metadata cleared. |
| [`decode_db_value_bytes`](#decode_db_value_bytes) | function | Decodes a standalone Cap'n `DbValue` message. |
| [`decode_execute_request_bytes`](#decode_execute_request_bytes) | function | Decodes a standalone Cap'n `ExecuteRequest` message. |
| [`decode_execute_result_reply_bytes`](#decode_execute_result_reply_bytes) | function | Decodes a standalone Cap'n `ExecuteAtomicReply` message. |
| [`encoded_db_value_bytes`](#encoded_db_value_bytes) | function | Encodes a standalone Cap'n `DbValue` message (unpacked stream). |
| [`encoded_execute_reply_bytes`](#encoded_execute_reply_bytes) | function | Encodes a standalone Cap'n `ExecuteReply` message (unpacked stream). |
| [`encoded_execute_request_bytes`](#encoded_execute_request_bytes) | function | Encodes a standalone Cap'n `ExecuteRequest` message (unpacked stream). |
| [`encoded_execute_result_reply_bytes`](#encoded_execute_result_reply_bytes) | function | Encodes a standalone Cap'n `ExecuteAtomicReply` (`ok` or `err`). |
| [`encoded_statement_result_bytes`](#encoded_statement_result_bytes) | function | Encodes a standalone Cap'n `StatementResult` message (unpacked stream). |
| [`negotiate_rpc_features`](#negotiate_rpc_features) | function | Intersects host-offered and guest-accepted feature lists. |
| [`read_all`](#read_all) | function | Reads a small transferred body, stopping at `crate::MAX_SCALAR_BYTES`. |
| [`stream_copy_keys`](#stream_copy_keys) | function | Copies `from` → `to` through granted source/destination streams. |
| [`byte_source_from_async_read`](#byte_source_from_async_read) | function | Returns a `ByteSource` client that pulls from `reader`. |
| [`connect_plugin`](#connect_plugin) | function | Connects as the client side of a two-party vat. |
| [`pull_byte_source_to_writer`](#pull_byte_source_to_writer) | function | Pulls `source` into `writer` using bounded windows. |
| [`serve_plugin`](#serve_plugin) | function | Serves `plugin` as the bootstrap object on a two-party vat over `reader`/`writer`. |
| [`serve_plugin_stdio`](#serve_plugin_stdio) | function | Serves `plugin` on stdin/stdout. Must run on a current-thread runtime + `LocalSet`. |
| [`MAX_PLUGIN_MIGRATION_ID_BYTES`](#max_plugin_migration_id_bytes) | constant | Maximum UTF-8 bytes of one plugin-chosen migration id. |
| [`PLUGIN_MIGRATIONS_TABLE`](#plugin_migrations_table) | constant | Host-private plugin-binding migration journal (not a plugin-owned table). |
| [`PHYSICAL_STAR_COLUMN`](#physical_star_column) | constant | Column name recorded for `SELECT *` / `alias.*` / `RETURNING *` on a physical table. |
| [`D1_PHYSICAL_LIKE_GLOB_PATTERN_BYTES`](#d1_physical_like_glob_pattern_bytes) | constant | Physical D1 LIKE/GLOB pattern cap in bytes. |
| [`D1_PORTABLE_LIKE_PATTERN_BYTES`](#d1_portable_like_pattern_bytes) | constant | Conservative portable LIKE pattern cap for D1 while LIKE lowers to GLOB. |
| [`LIKE_GLOB_PATTERN_WRAP_BYTES`](#like_glob_pattern_wrap_bytes) | constant | Bytes added around a LIKE pattern by `LIKE_GLOB_WRAP_PREFIX` / `LIKE_GLOB_WRAP_SUFFIX` (the pattern itself is unchanged). |
| [`LIKE_GLOB_REWRITE_OVERHEAD`](#like_glob_rewrite_overhead) | constant | Worst-case extra bytes per `LIKE` / `NOT LIKE` after sqlite-family GLOB wrapping. |
| [`LIKE_GLOB_WRAP_PREFIX`](#like_glob_wrap_prefix) | constant | Prefix of the sqlite-family LIKE→GLOB `replace(...)` wrap (before the pattern). |
| [`LIKE_GLOB_WRAP_SUFFIX`](#like_glob_wrap_suffix) | constant | Suffix of the sqlite-family LIKE→GLOB `replace(...)` wrap (after the pattern). |
| [`LIKE_TO_GLOB_KEYWORD_EXTRA`](#like_to_glob_keyword_extra) | constant | Extra byte when `LIKE` is immediately followed by its pattern (no trivia): the rewrite emits `GLOB ` (5) in place of `LIKE` (4). |
| [`SQLITE_FAMILY_DIV_MOD_NULLIF_EXTRA`](#sqlite_family_div_mod_nullif_extra) | constant | Extra bytes for one `/` or `%` rewritten to `NULLIF(operand, 0)`. |
| [`SQLITE_FAMILY_INSERT_OR_IGNORE_MAX_EXTRA`](#sqlite_family_insert_or_ignore_max_extra) | constant | Conservative extra bytes for one `INSERT OR IGNORE` unique-conflict rewrite (conflict clause plus optional `SELECT` source wrap). |
| [`FEATURE_SCALAR_LIMITS`](#feature_scalar_limits) | constant | Guest honors scalar / stream-window / list-page caps. |
| [`FEATURE_STORAGE_COPY`](#feature_storage_copy) | constant | Guest implements server-side `super::Destination::copy`. |
| [`FEATURE_STREAMS`](#feature_streams) | constant | Media moves through transferred `crate::roles::ByteRange` / `ByteSource` streams. |
| [`MAX_EVENT_PAYLOAD_BYTES`](#max_event_payload_bytes) | constant | Maximum decoded size of a domain-event scalar payload (not a stream). |
| [`MAX_KV_KEY_BYTES`](#max_kv_key_bytes) | constant | Longest `KvNamespace` key in UTF-8 bytes. |
| [`MAX_KV_METADATA_BYTES`](#max_kv_metadata_bytes) | constant | Largest `KvNamespace` metadata JSON text in bytes. |
| [`MAX_KV_VALUE_BYTES`](#max_kv_value_bytes) | constant | Largest `KvNamespace` value in bytes (a `[[kv_namespaces]]` row may lower its own cap but never raise it past this). |
| [`MAX_LIST_PAGE`](#max_list_page) | constant | Maximum objects in one `Destination.list` page. |
| [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops) | constant | Maximum already-separated operations in one plugin-owned migration. |
| [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes) | constant | Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one `databaseMigrations` registration (Cap'n Proto framing is not counted). |
| [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops) | constant | Maximum already-separated operations across one `databaseMigrations` registration (independent of the per-migration cap and of id+SQL UTF-8 bytes). |
| [`MAX_SCALAR_BYTES`](#max_scalar_bytes) | constant | Maximum decoded size of an ordinary RPC scalar value (not a stream window). |
| [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) | constant | Maximum bytes returned by one `ByteSource.pull` (flow-control window). |
| [`PRODUCT_API_VERSION`](#product_api_version) | constant | Product ABI version (`apiVersion` / `plugin.toml` `api_version`). |
| [`JSON_MEDIA_TYPE`](#json_media_type) | constant | Media type of a JSON `ExtensibleConfig` payload. |
| [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes) | constant | Maximum checkpoint payload size (bytes). |
| [`PLUGIN_TOML_SCHEMA_JSON`](#plugin_toml_schema_json) | constant | Embedded JSON Schema for install `plugin.toml` files (shared with language SDK author tools and `bookclerk plugins` validation). |

## Re-exports

- [`backup_ops::AdapterBackupOps`](backup_ops.md#adapterbackupops)
- [`backup_ops::SharedAdapterBackupOps`](backup_ops.md#sharedadapterbackupops)
- [`db::binding_values_from_adapter_config`](db.md#binding_values_from_adapter_config)
- [`db::database_adapter_config_from_bindings`](db.md#database_adapter_config_from_bindings)
- [`db::binding_values_from_params`](db.md#binding_values_from_params)
- [`db::connect_params_from_bindings`](db.md#connect_params_from_bindings)
- [`db::DbConnectParams`](db.md#dbconnectparams)
- [`db_execute::lowered_statement_preflight_len_proven`](db_execute.md#lowered_statement_preflight_len_proven)
- [`db_execute::lowered_statement_preflight_len`](db_execute.md#lowered_statement_preflight_len)
- [`db_execute::lowered_statement_upper_bound_len`](db_execute.md#lowered_statement_upper_bound_len)
- [`db_execute::sql_payload_bytes`](db_execute.md#sql_payload_bytes)
- [`db_execute::sql_payload_exceeds`](db_execute.md#sql_payload_exceeds)
- [`db_execute::DbBootstrap`](db_execute.md#dbbootstrap)
- [`db_execute::DbCapabilities`](db_execute.md#dbcapabilities)
- [`db_execute::DbColumn`](db_execute.md#dbcolumn)
- [`db_execute::DbIdentityHighWater`](db_execute.md#dbidentityhighwater)
- [`db_execute::DbPlanStatementKind`](db_execute.md#dbplanstatementkind)
- [`db_execute::DbResultSelection`](db_execute.md#dbresultselection)
- [`db_execute::DbRow`](db_execute.md#dbrow)
- [`db_execute::DbTiming`](db_execute.md#dbtiming)
- [`db_execute::ExecuteReply`](db_execute.md#executereply)
- [`db_execute::ExecuteRequest`](db_execute.md#executerequest)
- [`db_execute::IsolationReq`](db_execute.md#isolationreq)
- [`db_execute::StatementResult`](db_execute.md#statementresult)
- [`db_execute::TypedDbStatement`](db_execute.md#typeddbstatement)
- [`db_execute::D1_MAX_BINDS`](db_execute.md#d1_max_binds)
- [`db_execute::D1_MAX_FUNCTION_ARGS`](db_execute.md#d1_max_function_args)
- [`db_execute::D1_MAX_PAYLOAD_BYTES`](db_execute.md#d1_max_payload_bytes)
- [`db_execute::D1_MAX_SCHEMA_COLUMNS`](db_execute.md#d1_max_schema_columns)
- [`db_execute::D1_MAX_SQL_STATEMENT_BYTES`](db_execute.md#d1_max_sql_statement_bytes)
- [`db_execute::FIRST_PARTY_MAX_RESULT_BYTES`](db_execute.md#first_party_max_result_bytes)
- [`db_execute::FIRST_PARTY_MAX_RESULT_ROWS`](db_execute.md#first_party_max_result_rows)
- [`db_execute::FIRST_PARTY_MAX_STATEMENTS`](db_execute.md#first_party_max_statements)
- [`db_execute::HOST_MIN_BINDS`](db_execute.md#host_min_binds)
- [`db_execute::HOST_MIN_CELL_BYTES`](db_execute.md#host_min_cell_bytes)
- [`db_execute::HOST_MIN_FUNCTION_ARGS`](db_execute.md#host_min_function_args)
- [`db_execute::HOST_MIN_PATTERN_BYTES`](db_execute.md#host_min_pattern_bytes)
- [`db_execute::HOST_MIN_PAYLOAD_BYTES`](db_execute.md#host_min_payload_bytes)
- [`db_execute::HOST_MIN_RESULT_BYTES`](db_execute.md#host_min_result_bytes)
- [`db_execute::HOST_MIN_RESULT_ROWS`](db_execute.md#host_min_result_rows)
- [`db_execute::HOST_MIN_SCHEMA_COLUMNS`](db_execute.md#host_min_schema_columns)
- [`db_execute::HOST_MIN_STATEMENTS`](db_execute.md#host_min_statements)
- [`db_execute::POSTGRES_MAX_BINDS`](db_execute.md#postgres_max_binds)
- [`db_execute::POSTGRES_MAX_FUNCTION_ARGS`](db_execute.md#postgres_max_function_args)
- [`db_execute::POSTGRES_MAX_SCHEMA_COLUMNS`](db_execute.md#postgres_max_schema_columns)
- [`db_execute::SQLITE_MAX_BINDS`](db_execute.md#sqlite_max_binds)
- [`db_execute::SQLITE_MAX_FUNCTION_ARGS`](db_execute.md#sqlite_max_function_args)
- [`db_execute::SQLITE_MAX_PATTERN_BYTES`](db_execute.md#sqlite_max_pattern_bytes)
- [`db_execute::SQLITE_MAX_SCHEMA_COLUMNS`](db_execute.md#sqlite_max_schema_columns)
- [`db_execute::SQL_CONTRACT_VERSION`](db_execute.md#sql_contract_version)
- [`db_value::db_type_from_declared`](db_value.md#db_type_from_declared)
- [`db_value::normalize_db_value_for_column`](db_value.md#normalize_db_value_for_column)
- [`db_value::DbType`](db_value.md#dbtype)
- [`db_value::DbValue`](db_value.md#dbvalue)
- [`error::PluginError`](error.md#pluginerror)
- [`error::PluginErrorCode`](error.md#pluginerrorcode)
- [`error::Result`](error.md#result)
- [`guest_sql::authorize_guest_sql_policy`](guest_sql.md#authorize_guest_sql_policy)
- [`guest_sql::guest_statement_kind`](guest_sql.md#guest_statement_kind)
- [`guest_sql::is_reserved_binding_name`](guest_sql.md#is_reserved_binding_name)
- [`guest_sql::parse_guest_sql_refs`](guest_sql.md#parse_guest_sql_refs)
- [`guest_sql::returning_single_row_proven`](guest_sql.md#returning_single_row_proven)
- [`guest_sql::statement_is_ddl`](guest_sql.md#statement_is_ddl)
- [`guest_sql::validate_guest_execute_request`](guest_sql.md#validate_guest_execute_request)
- [`guest_sql::validate_guest_execute_request_for_policy`](guest_sql.md#validate_guest_execute_request_for_policy)
- [`guest_sql::validate_sql_v1_grammar`](guest_sql.md#validate_sql_v1_grammar)
- [`guest_sql::GuestSqlPolicy`](guest_sql.md#guestsqlpolicy)
- [`guest_sql::GuestSqlRefs`](guest_sql.md#guestsqlrefs)
- [`guest_sql::BOOKCLERK_RESERVED_PREFIX`](guest_sql.md#bookclerk_reserved_prefix)
- [`methods::METHOD_NAMES`](methods.md#method_names)
- [`sql_desugar::desugar_canonical_sql`](sql_desugar.md#desugar_canonical_sql)
- [`sql_desugar::desugar_execute_request`](sql_desugar.md#desugar_execute_request)
- [`sql_types::apply_schema_action_to_env`](sql_types.md#apply_schema_action_to_env)
- [`sql_types::apply_schema_sql_to_env`](sql_types.md#apply_schema_sql_to_env)
- [`sql_types::catalog_companions`](sql_types.md#catalog_companions)
- [`sql_types::catalog_page_statement`](sql_types.md#catalog_page_statement)
- [`sql_types::parse_create_index_sql`](sql_types.md#parse_create_index_sql)
- [`sql_types::parse_create_table_schema`](sql_types.md#parse_create_table_schema)
- [`sql_types::parse_drop_index_name`](sql_types.md#parse_drop_index_name)
- [`sql_types::parse_drop_table_name`](sql_types.md#parse_drop_table_name)
- [`sql_types::postgres_identity_function_name`](sql_types.md#postgres_identity_function_name)
- [`sql_types::postgres_identity_object_digest`](sql_types.md#postgres_identity_object_digest)
- [`sql_types::postgres_identity_trigger_name`](sql_types.md#postgres_identity_trigger_name)
- [`sql_types::require_sql_v1_helper_arity`](sql_types.md#require_sql_v1_helper_arity)
- [`sql_types::reserved_catalog_relation_missing`](sql_types.md#reserved_catalog_relation_missing)
- [`sql_types::sql_catalog_create_table_sql`](sql_types.md#sql_catalog_create_table_sql)
- [`sql_types::sql_catalog_page_rows`](sql_types.md#sql_catalog_page_rows)
- [`sql_types::sql_ddl_create_table_sql`](sql_types.md#sql_ddl_create_table_sql)
- [`sql_types::sql_host_bookkeeping_type_env`](sql_types.md#sql_host_bookkeeping_type_env)
- [`sql_types::sql_type_env_from_canonical_ddl`](sql_types.md#sql_type_env_from_canonical_ddl)
- [`sql_types::sql_type_env_from_canonical_statements`](sql_types.md#sql_type_env_from_canonical_statements)
- [`sql_types::sql_v1_helper_arity`](sql_types.md#sql_v1_helper_arity)
- [`sql_types::sql_v1_helper_arity_ok`](sql_types.md#sql_v1_helper_arity_ok)
- [`sql_types::sql_v1_ident_in_bounds`](sql_types.md#sql_v1_ident_in_bounds)
- [`sql_types::statement_sql_hash`](sql_types.md#statement_sql_hash)
- [`sql_types::typecheck_create_index_sql`](sql_types.md#typecheck_create_index_sql)
- [`sql_types::typecheck_execute_request`](sql_types.md#typecheck_execute_request)
- [`sql_types::ColumnReference`](sql_types.md#columnreference)
- [`sql_types::CreateIndexSchema`](sql_types.md#createindexschema)
- [`sql_types::CreateTableSchema`](sql_types.md#createtableschema)
- [`sql_types::SqlType`](sql_types.md#sqltype)
- [`sql_types::SqlTypeEnv`](sql_types.md#sqltypeenv)
- [`sql_types::INSERT_SELECT_WRAP_ALIAS`](sql_types.md#insert_select_wrap_alias)
- [`sql_types::POSTGRES_IDENT_FN_PREFIX`](sql_types.md#postgres_ident_fn_prefix)
- [`sql_types::POSTGRES_IDENT_TRIGGER_PREFIX`](sql_types.md#postgres_ident_trigger_prefix)
- [`sql_types::SQL_CATALOG_TABLE`](sql_types.md#sql_catalog_table)
- [`sql_types::SQL_DDL_TABLE`](sql_types.md#sql_ddl_table)
- [`sql_types::SQL_IDENTITY_TABLE`](sql_types.md#sql_identity_table)
- [`sql_types::SQL_SCHEMA_TABLE`](sql_types.md#sql_schema_table)
- [`sql_types::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#sql_v1_json_object_max_args)
- [`sql_types::SQL_V1_MAX_IDENT_BYTES`](sql_types.md#sql_v1_max_ident_bytes)
- [`sql_types::catalog_companions_for_action`](sql_types.md#catalog_companions_for_action)
- [`sql_types::sql_schema_create_table_sql`](sql_types.md#sql_schema_create_table_sql)
- [`sql_types::typecheck_execute_request_proofs`](sql_types.md#typecheck_execute_request_proofs)
- [`settings_schema::is_valid_setting_key`](settings_schema.md#is_valid_setting_key)
- [`settings_schema::EffectiveConfig`](settings_schema.md#effectiveconfig)
- [`settings_schema::SettingKind`](settings_schema.md#settingkind)
- [`settings_schema::SettingSpec`](settings_schema.md#settingspec)
- [`settings_schema::SettingValueSpec`](settings_schema.md#settingvaluespec)
- [`settings_schema::SettingsSchema`](settings_schema.md#settingsschema)
- [`settings_schema::MAX_SETTINGS`](settings_schema.md#max_settings)
- [`settings_schema::MAX_SETTING_STRING_BYTES`](settings_schema.md#max_setting_string_bytes)
- [`settings_schema::SETTINGS_SCHEMA_VERSION`](settings_schema.md#settings_schema_version)
- [`generated::Entrypoint`](generated.md#entrypoint)
- [`generated::PortalAuthMode`](generated.md#portalauthmode)
- [`generated::CliArgKind`](generated.md#cliargkind)
- [`generated::CatalogSort`](generated.md#catalogsort)
- [`generated::CatalogField`](generated.md#catalogfield)
- [`generated::Abridgement`](generated.md#abridgement)
- [`generated::EventConsumerSpec`](generated.md#eventconsumerspec)
- [`generated::PluginCapabilities`](generated.md#plugincapabilities)
- [`generated::Brand`](generated.md#brand)
- [`generated::ConfigOption`](generated.md#configoption)
- [`generated::ConfigOptionValue`](generated.md#configoptionvalue)
- [`generated::CliSchema`](generated.md#clischema)
- [`generated::CliCommandSpec`](generated.md#clicommandspec)
- [`generated::CliArgSpec`](generated.md#cliargspec)
- [`generated::CliArg`](generated.md#cliarg)
- [`generated::CliInvokeParams`](generated.md#cliinvokeparams)
- [`generated::CliInvokeResult`](generated.md#cliinvokeresult)
- [`generated::DatabaseAdapterConfig`](generated.md#databaseadapterconfig)
- [`generated::DiagnoseResult`](generated.md#diagnoseresult)
- [`generated::SourceAccount`](generated.md#sourceaccount)
- [`generated::LoginParams`](generated.md#loginparams)
- [`generated::LoginResult`](generated.md#loginresult)
- [`generated::LoginStartResult`](generated.md#loginstartresult)
- [`generated::LoginCompleteParams`](generated.md#logincompleteparams)
- [`generated::AccountCredential`](generated.md#accountcredential)
- [`generated::ScanParams`](generated.md#scanparams)
- [`generated::ScanBook`](generated.md#scanbook)
- [`generated::ScanSummary`](generated.md#scansummary)
- [`generated::FetchOptions`](generated.md#fetchoptions)
- [`generated::FetchTitleParams`](generated.md#fetchtitleparams)
- [`generated::PlainPart`](generated.md#plainpart)
- [`generated::ChapterMarker`](generated.md#chaptermarker)
- [`generated::PlainFetch`](generated.md#plainfetch)
- [`generated::SourceAccounts`](generated.md#sourceaccounts)
- [`generated::SearchCatalogParams`](generated.md#searchcatalogparams)
- [`generated::ExpandCandidatesParams`](generated.md#expandcandidatesparams)
- [`generated::PurchaseHintParams`](generated.md#purchasehintparams)
- [`generated::ListDealsParams`](generated.md#listdealsparams)
- [`generated::CatalogDetailParams`](generated.md#catalogdetailparams)
- [`generated::RevokeAccountParams`](generated.md#revokeaccountparams)
- [`generated::CatalogHit`](generated.md#cataloghit)
- [`generated::CatalogHits`](generated.md#cataloghits)
- [`generated::CatalogDetail`](generated.md#catalogdetail)
- [`generated::PurchaseHint`](generated.md#purchasehint)
- [`generated::PurchaseHintResult`](generated.md#purchasehintresult)
- [`generated::ScanLibraryParams`](generated.md#scanlibraryparams)
- [`generated::AuthenticateUserParams`](generated.md#authenticateuserparams)
- [`generated::ExternalUser`](generated.md#externaluser)
- [`generated::EventPollResult`](generated.md#eventpollresult)
- [`generated::ListeningProgress`](generated.md#listeningprogress)
- [`generated::SyncListeningResult`](generated.md#synclisteningresult)
- [`generated::PluginEvent`](generated.md#pluginevent)
- [`generated::PublishOk`](generated.md#publishok)
- [`generated::Invocation`](generated.md#invocation)
- [`generated::QueueMessage`](generated.md#queuemessage)
- [`generated::QueueBatch`](generated.md#queuebatch)
- [`generated::QueueRetry`](generated.md#queueretry)
- [`generated::QueueResult`](generated.md#queueresult)
- [`generated::QueueBatchOk`](generated.md#queuebatchok)
- [`generated::QueueSendMessage`](generated.md#queuesendmessage)
- [`generated::QueueSendOk`](generated.md#queuesendok)
- [`generated::KvPutOptions`](generated.md#kvputoptions)
- [`generated::KvGetOk`](generated.md#kvgetok)
- [`generated::KvPutOk`](generated.md#kvputok)
- [`generated::KvDeleteOk`](generated.md#kvdeleteok)
- [`generated::KvListOptions`](generated.md#kvlistoptions)
- [`generated::KvListKey`](generated.md#kvlistkey)
- [`generated::KvListOk`](generated.md#kvlistok)
- [`generated::write_event_consumer_spec`](generated.md#write_event_consumer_spec)
- [`generated::read_event_consumer_spec`](generated.md#read_event_consumer_spec)
- [`generated::write_plugin_capabilities`](generated.md#write_plugin_capabilities)
- [`generated::read_plugin_capabilities`](generated.md#read_plugin_capabilities)
- [`generated::write_brand`](generated.md#write_brand)
- [`generated::read_brand`](generated.md#read_brand)
- [`generated::write_config_option`](generated.md#write_config_option)
- [`generated::read_config_option`](generated.md#read_config_option)
- [`generated::write_config_option_value`](generated.md#write_config_option_value)
- [`generated::read_config_option_value`](generated.md#read_config_option_value)
- [`generated::write_cli_schema`](generated.md#write_cli_schema)
- [`generated::read_cli_schema`](generated.md#read_cli_schema)
- [`generated::write_cli_command_spec`](generated.md#write_cli_command_spec)
- [`generated::read_cli_command_spec`](generated.md#read_cli_command_spec)
- [`generated::write_cli_arg_spec`](generated.md#write_cli_arg_spec)
- [`generated::read_cli_arg_spec`](generated.md#read_cli_arg_spec)
- [`generated::write_cli_arg`](generated.md#write_cli_arg)
- [`generated::read_cli_arg`](generated.md#read_cli_arg)
- [`generated::write_cli_invoke_params`](generated.md#write_cli_invoke_params)
- [`generated::read_cli_invoke_params`](generated.md#read_cli_invoke_params)
- [`generated::write_cli_invoke_result`](generated.md#write_cli_invoke_result)
- [`generated::read_cli_invoke_result`](generated.md#read_cli_invoke_result)
- [`generated::write_cli_schema_reply`](generated.md#write_cli_schema_reply)
- [`generated::read_cli_schema_reply`](generated.md#read_cli_schema_reply)
- [`generated::write_cli_invoke_reply`](generated.md#write_cli_invoke_reply)
- [`generated::read_cli_invoke_reply`](generated.md#read_cli_invoke_reply)
- [`generated::write_database_adapter_config`](generated.md#write_database_adapter_config)
- [`generated::read_database_adapter_config`](generated.md#read_database_adapter_config)
- [`generated::write_diagnose_result`](generated.md#write_diagnose_result)
- [`generated::read_diagnose_result`](generated.md#read_diagnose_result)
- [`generated::write_diagnose_reply`](generated.md#write_diagnose_reply)
- [`generated::read_diagnose_reply`](generated.md#read_diagnose_reply)
- [`generated::write_source_account`](generated.md#write_source_account)
- [`generated::read_source_account`](generated.md#read_source_account)
- [`generated::write_login_params`](generated.md#write_login_params)
- [`generated::read_login_params`](generated.md#read_login_params)
- [`generated::write_login_result`](generated.md#write_login_result)
- [`generated::read_login_result`](generated.md#read_login_result)
- [`generated::write_login_start_result`](generated.md#write_login_start_result)
- [`generated::read_login_start_result`](generated.md#read_login_start_result)
- [`generated::write_login_complete_params`](generated.md#write_login_complete_params)
- [`generated::read_login_complete_params`](generated.md#read_login_complete_params)
- [`generated::write_login_reply`](generated.md#write_login_reply)
- [`generated::read_login_reply`](generated.md#read_login_reply)
- [`generated::write_login_start_reply`](generated.md#write_login_start_reply)
- [`generated::read_login_start_reply`](generated.md#read_login_start_reply)
- [`generated::write_account_credential`](generated.md#write_account_credential)
- [`generated::read_account_credential`](generated.md#read_account_credential)
- [`generated::write_scan_params`](generated.md#write_scan_params)
- [`generated::read_scan_params`](generated.md#read_scan_params)
- [`generated::write_scan_book`](generated.md#write_scan_book)
- [`generated::read_scan_book`](generated.md#read_scan_book)
- [`generated::write_scan_summary`](generated.md#write_scan_summary)
- [`generated::read_scan_summary`](generated.md#read_scan_summary)
- [`generated::write_scan_reply`](generated.md#write_scan_reply)
- [`generated::read_scan_reply`](generated.md#read_scan_reply)
- [`generated::write_fetch_options`](generated.md#write_fetch_options)
- [`generated::read_fetch_options`](generated.md#read_fetch_options)
- [`generated::write_fetch_title_params`](generated.md#write_fetch_title_params)
- [`generated::read_fetch_title_params`](generated.md#read_fetch_title_params)
- [`generated::write_plain_part`](generated.md#write_plain_part)
- [`generated::read_plain_part`](generated.md#read_plain_part)
- [`generated::write_chapter_marker`](generated.md#write_chapter_marker)
- [`generated::read_chapter_marker`](generated.md#read_chapter_marker)
- [`generated::write_plain_fetch`](generated.md#write_plain_fetch)
- [`generated::read_plain_fetch`](generated.md#read_plain_fetch)
- [`generated::write_fetch_title_reply`](generated.md#write_fetch_title_reply)
- [`generated::read_fetch_title_reply`](generated.md#read_fetch_title_reply)
- [`generated::write_source_accounts`](generated.md#write_source_accounts)
- [`generated::read_source_accounts`](generated.md#read_source_accounts)
- [`generated::write_source_accounts_reply`](generated.md#write_source_accounts_reply)
- [`generated::read_source_accounts_reply`](generated.md#read_source_accounts_reply)
- [`generated::write_search_catalog_params`](generated.md#write_search_catalog_params)
- [`generated::read_search_catalog_params`](generated.md#read_search_catalog_params)
- [`generated::write_expand_candidates_params`](generated.md#write_expand_candidates_params)
- [`generated::read_expand_candidates_params`](generated.md#read_expand_candidates_params)
- [`generated::write_purchase_hint_params`](generated.md#write_purchase_hint_params)
- [`generated::read_purchase_hint_params`](generated.md#read_purchase_hint_params)
- [`generated::write_list_deals_params`](generated.md#write_list_deals_params)
- [`generated::read_list_deals_params`](generated.md#read_list_deals_params)
- [`generated::write_catalog_detail_params`](generated.md#write_catalog_detail_params)
- [`generated::read_catalog_detail_params`](generated.md#read_catalog_detail_params)
- [`generated::write_revoke_account_params`](generated.md#write_revoke_account_params)
- [`generated::read_revoke_account_params`](generated.md#read_revoke_account_params)
- [`generated::write_catalog_hit`](generated.md#write_catalog_hit)
- [`generated::read_catalog_hit`](generated.md#read_catalog_hit)
- [`generated::write_catalog_hits`](generated.md#write_catalog_hits)
- [`generated::read_catalog_hits`](generated.md#read_catalog_hits)
- [`generated::write_catalog_hits_reply`](generated.md#write_catalog_hits_reply)
- [`generated::read_catalog_hits_reply`](generated.md#read_catalog_hits_reply)
- [`generated::write_catalog_detail`](generated.md#write_catalog_detail)
- [`generated::read_catalog_detail`](generated.md#read_catalog_detail)
- [`generated::write_catalog_detail_reply`](generated.md#write_catalog_detail_reply)
- [`generated::read_catalog_detail_reply`](generated.md#read_catalog_detail_reply)
- [`generated::write_purchase_hint`](generated.md#write_purchase_hint)
- [`generated::read_purchase_hint`](generated.md#read_purchase_hint)
- [`generated::write_purchase_hint_result`](generated.md#write_purchase_hint_result)
- [`generated::read_purchase_hint_result`](generated.md#read_purchase_hint_result)
- [`generated::write_purchase_hint_reply`](generated.md#write_purchase_hint_reply)
- [`generated::read_purchase_hint_reply`](generated.md#read_purchase_hint_reply)
- [`generated::write_scan_library_params`](generated.md#write_scan_library_params)
- [`generated::read_scan_library_params`](generated.md#read_scan_library_params)
- [`generated::write_authenticate_user_params`](generated.md#write_authenticate_user_params)
- [`generated::read_authenticate_user_params`](generated.md#read_authenticate_user_params)
- [`generated::write_external_user`](generated.md#write_external_user)
- [`generated::read_external_user`](generated.md#read_external_user)
- [`generated::write_external_user_reply`](generated.md#write_external_user_reply)
- [`generated::read_external_user_reply`](generated.md#read_external_user_reply)
- [`generated::write_event_poll_result`](generated.md#write_event_poll_result)
- [`generated::read_event_poll_result`](generated.md#read_event_poll_result)
- [`generated::write_event_poll_reply`](generated.md#write_event_poll_reply)
- [`generated::read_event_poll_reply`](generated.md#read_event_poll_reply)
- [`generated::write_listening_progress`](generated.md#write_listening_progress)
- [`generated::read_listening_progress`](generated.md#read_listening_progress)
- [`generated::write_sync_listening_result`](generated.md#write_sync_listening_result)
- [`generated::read_sync_listening_result`](generated.md#read_sync_listening_result)
- [`generated::write_sync_listening_reply`](generated.md#write_sync_listening_reply)
- [`generated::read_sync_listening_reply`](generated.md#read_sync_listening_reply)
- [`generated::write_plugin_event`](generated.md#write_plugin_event)
- [`generated::read_plugin_event`](generated.md#read_plugin_event)
- [`generated::write_publish_ok`](generated.md#write_publish_ok)
- [`generated::read_publish_ok`](generated.md#read_publish_ok)
- [`generated::write_publish_reply`](generated.md#write_publish_reply)
- [`generated::read_publish_reply`](generated.md#read_publish_reply)
- [`generated::write_invocation`](generated.md#write_invocation)
- [`generated::read_invocation`](generated.md#read_invocation)
- [`generated::write_queue_message`](generated.md#write_queue_message)
- [`generated::read_queue_message`](generated.md#read_queue_message)
- [`generated::write_queue_batch`](generated.md#write_queue_batch)
- [`generated::read_queue_batch`](generated.md#read_queue_batch)
- [`generated::write_queue_retry`](generated.md#write_queue_retry)
- [`generated::read_queue_retry`](generated.md#read_queue_retry)
- [`generated::write_queue_result`](generated.md#write_queue_result)
- [`generated::read_queue_result`](generated.md#read_queue_result)
- [`generated::write_queue_batch_ok`](generated.md#write_queue_batch_ok)
- [`generated::read_queue_batch_ok`](generated.md#read_queue_batch_ok)
- [`generated::write_queue_batch_reply`](generated.md#write_queue_batch_reply)
- [`generated::read_queue_batch_reply`](generated.md#read_queue_batch_reply)
- [`generated::write_queue_send_message`](generated.md#write_queue_send_message)
- [`generated::read_queue_send_message`](generated.md#read_queue_send_message)
- [`generated::write_queue_send_ok`](generated.md#write_queue_send_ok)
- [`generated::read_queue_send_ok`](generated.md#read_queue_send_ok)
- [`generated::write_queue_send_reply`](generated.md#write_queue_send_reply)
- [`generated::read_queue_send_reply`](generated.md#read_queue_send_reply)
- [`generated::write_kv_put_options`](generated.md#write_kv_put_options)
- [`generated::read_kv_put_options`](generated.md#read_kv_put_options)
- [`generated::write_kv_get_ok`](generated.md#write_kv_get_ok)
- [`generated::read_kv_get_ok`](generated.md#read_kv_get_ok)
- [`generated::write_kv_get_reply`](generated.md#write_kv_get_reply)
- [`generated::read_kv_get_reply`](generated.md#read_kv_get_reply)
- [`generated::write_kv_put_ok`](generated.md#write_kv_put_ok)
- [`generated::read_kv_put_ok`](generated.md#read_kv_put_ok)
- [`generated::write_kv_delete_ok`](generated.md#write_kv_delete_ok)
- [`generated::read_kv_delete_ok`](generated.md#read_kv_delete_ok)
- [`generated::write_kv_put_reply`](generated.md#write_kv_put_reply)
- [`generated::read_kv_put_reply`](generated.md#read_kv_put_reply)
- [`generated::write_kv_delete_reply`](generated.md#write_kv_delete_reply)
- [`generated::read_kv_delete_reply`](generated.md#read_kv_delete_reply)
- [`generated::write_kv_list_options`](generated.md#write_kv_list_options)
- [`generated::read_kv_list_options`](generated.md#read_kv_list_options)
- [`generated::write_kv_list_key`](generated.md#write_kv_list_key)
- [`generated::read_kv_list_key`](generated.md#read_kv_list_key)
- [`generated::write_kv_list_ok`](generated.md#write_kv_list_ok)
- [`generated::read_kv_list_ok`](generated.md#read_kv_list_ok)
- [`generated::write_kv_list_reply`](generated.md#write_kv_list_reply)
- [`generated::read_kv_list_reply`](generated.md#read_kv_list_reply)
- [`generated::READ_ANY_COUNT`](generated.md#read_any_count)
- [`generated::read_any_message`](generated.md#read_any_message)
- [`generated::RoundTripFailure`](generated.md#roundtripfailure)
- [`generated::roundtrip_any_message`](generated.md#roundtrip_any_message)
- [`kind::S3CredentialsDto`](kind.md#s3credentialsdto)
- [`kind::OutputS3ContextDto`](kind.md#outputs3contextdto)
- [`kind::OutputLocalContextDto`](kind.md#outputlocalcontextdto)

## Modules

- [`backup_ops`](backup_ops.md): Host-facing backup/restore primitives (Send + Sync).
- [`db`](db.md): Database-adapter binding payloads (host connect params + public adapter config).
- [`db_execute`](db_execute.md): Typed Cap'n database data-plane (`ExecuteRequest` / `ExecuteReply`) and control-plane (`DbCapabilities`) mirrors of `plugin.capnp`.
- [`db_value`](db_value.md): Universal Cap'n database value domain (`DbValue`).
- [`error`](error.md): Plugin error model shared by host and guests.
- [`generated`](generated.md): GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.
- [`guest_sql`](guest_sql.md): Host-side grammar and scope checks for guest-authored typed SQL.
- [`json_bytes`](json_bytes.md): Serde helpers projecting Cap'n Proto `Data` fields as base64 text in JSON.
- [`kind`](kind.md): Output-plugin configuration DTOs carried as `application/json` inside `crate::ExtensibleConfig` (`Bindings.config` for the `storage` entrypoint).
- [`methods`](methods.md): Role capability method names (camelCase on the wire).
- [`package_version`](package_version.md): `plugin.toml` `version` as a semantic version.
- [`settings_schema`](settings_schema.md): Operator-facing plugin settings schema (`[[settings]]` in `plugin.toml`).
- [`sql_desugar`](sql_desugar.md): Host semantic desugars for canonical Bookclerk SQL.
- [`sql_types`](sql_types.md): Bookclerk SQL v1 types, CAST matrix, and fail-closed expression checking.
- [`plugin_capnp`](plugin_capnp.md): Generated Cap'n Proto RPC interfaces (`schema/plugin.capnp`).
- [`plugin_host_capnp`](plugin_host_capnp.md): Host-private Cap'n Proto RPC interfaces (`schema/plugin_host.capnp`).

## Traits

### AdapterTransaction

```rust
pub trait AdapterTransaction {
    fn execute<'life0, 'async_trait>(
        &'life0 self,
        request: AdapterExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn commit<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn rollback<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn export_identity<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn import_identity<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _rows: &'life1 [DbIdentityHighWater],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_user_relations<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn prepare_unit_restore<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn drop_user_relations<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _names: &'life1 [String],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn assert_restore_constraints<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Host-only interactive transaction on an adapter connection (SeaORM proxy).

#### AdapterTransaction::execute

```rust
fn execute<'life0, 'async_trait>(
    &'life0 self,
    request: AdapterExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Typed statements on this open transaction (no second `BEGIN`).

Proofs are required ([`crate::AdapterExecuteRequest::require_proofs`](#adapterexecuterequestrequire_proofs)).

#### AdapterTransaction::commit

```rust
fn commit<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Commit.

#### AdapterTransaction::rollback

```rust
fn rollback<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Rollback.

#### AdapterTransaction::export_identity

```rust
fn export_identity<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Identity high-water from adapter catalogs on this transaction.

#### AdapterTransaction::import_identity

```rust
fn import_identity<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _rows: &'life1 [DbIdentityHighWater],
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Restore identity high-water into adapter catalogs on this transaction.

#### AdapterTransaction::list_user_relations

```rust
fn list_user_relations<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

User-visible relation names visible to this transaction.

#### AdapterTransaction::prepare_unit_restore

```rust
fn prepare_unit_restore<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Prepare this restore transaction (deferred FK checks).

#### AdapterTransaction::drop_user_relations

```rust
fn drop_user_relations<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _names: &'life1 [String],
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Drop named user relations on this transaction.

#### AdapterTransaction::assert_restore_constraints

```rust
fn assert_restore_constraints<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Fail closed when this restore transaction still has FK violations.

### HostAdapterDatabaseSession

```rust
pub trait HostAdapterDatabaseSession {
    fn begin<'life0, 'async_trait>(
        &'life0 self,
        isolation: IsolationReq,
    ) -> Pin<Box<dyn Future<Output = Result<Box<dyn AdapterTransaction>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn execute<'life0, 'async_trait>(
        &'life0 self,
        request: AdapterExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Host-only view of an open adapter session (`begin` for interactive txn).

#### HostAdapterDatabaseSession::begin

```rust
fn begin<'life0, 'async_trait>(
    &'life0 self,
    isolation: IsolationReq,
) -> Pin<Box<dyn Future<Output = Result<Box<dyn AdapterTransaction>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Opens a host-internal interactive transaction (SeaORM proxy).

#### HostAdapterDatabaseSession::execute

```rust
fn execute<'life0, 'async_trait>(
    &'life0 self,
    request: AdapterExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Typed atomic execute on this host-private session.

Implemented by: `HostAdapterDatabaseSessionClient`

### AdapterDatabaseSession

```rust
pub trait AdapterDatabaseSession {
    fn capabilities<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<DbCapabilities>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn bootstrap<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<DbBootstrap>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn execute<'life0, 'async_trait>(
        &'life0 self,
        request: AdapterExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn close<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn export_identity<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn import_identity<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _rows: &'life1 [DbIdentityHighWater],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list_user_relations<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn prepare_unit_restore<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn drop_user_relations<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _names: &'life1 [String],
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn assert_restore_constraints<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Host ↔ database adapter session (`capabilities` + typed `execute`).

#### AdapterDatabaseSession::capabilities

```rust
fn capabilities<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<DbCapabilities>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Typed SQL-contract advertisement.

#### AdapterDatabaseSession::bootstrap

```rust
fn bootstrap<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<DbBootstrap>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Bootstrap-only SeaORM proxy metadata for the open session.

#### AdapterDatabaseSession::execute

```rust
fn execute<'life0, 'async_trait>(
    &'life0 self,
    request: AdapterExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Typed atomic batch (`execute`). Canonical SQL plus required 1:1 proofs.

#### AdapterDatabaseSession::close

```rust
fn close<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Close the session.

#### AdapterDatabaseSession::export_identity

```rust
fn export_identity<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<DbIdentityHighWater>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Identity high-water from adapter catalogs.

#### AdapterDatabaseSession::import_identity

```rust
fn import_identity<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _rows: &'life1 [DbIdentityHighWater],
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Restore identity high-water into adapter catalogs.

#### AdapterDatabaseSession::list_user_relations

```rust
fn list_user_relations<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

User-visible relation names (excludes engine catalogs).

#### AdapterDatabaseSession::prepare_unit_restore

```rust
fn prepare_unit_restore<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Prepare an open restore transaction (deferred FK checks).

#### AdapterDatabaseSession::drop_user_relations

```rust
fn drop_user_relations<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _names: &'life1 [String],
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Drop named user relations (adapter-owned CASCADE / identity companions).

#### AdapterDatabaseSession::assert_restore_constraints

```rust
fn assert_restore_constraints<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Fail closed when the restore transaction still has FK violations.

### Cancellation

```rust
pub trait Cancellation {
    fn poll<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<bool>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Transport cancellation. SDKs project this into a locally created
`AbortSignal` (AbortSignal is not a serializable Workers RPC value).

#### Cancellation::poll

```rust
fn poll<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<bool>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Returns true when the host has cancelled this invocation.

Transport or capability failures must surface as `Err`. Callers must
fail closed (abort the invocation) rather than treating a failed poll as
"not cancelled".

Implemented by: `NeverCancel`

### ContentSource

```rust
pub trait ContentSource {
    fn login<'life0, 'async_trait>(
        &'life0 self,
        _params: LoginParams,
    ) -> Pin<Box<dyn Future<Output = Result<LoginResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn scan<'life0, 'async_trait>(
        &'life0 self,
        _params: ScanParams,
    ) -> Pin<Box<dyn Future<Output = Result<ScanSummary>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn fetch_title<'life0, 'async_trait>(
        &'life0 self,
        _params: FetchTitleParams,
    ) -> Pin<Box<dyn Future<Output = Result<PlainFetch>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn list_accounts<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn login_start<'life0, 'async_trait>(
        &'life0 self,
        _params: LoginParams,
    ) -> Pin<Box<dyn Future<Output = Result<LoginStartResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn login_complete<'life0, 'async_trait>(
        &'life0 self,
        _params: LoginCompleteParams,
    ) -> Pin<Box<dyn Future<Output = Result<LoginResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn search_catalog<'life0, 'async_trait>(
        &'life0 self,
        _params: SearchCatalogParams,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn expand_candidates<'life0, 'async_trait>(
        &'life0 self,
        _params: ExpandCandidatesParams,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn purchase_hint<'life0, 'async_trait>(
        &'life0 self,
        _params: PurchaseHintParams,
    ) -> Pin<Box<dyn Future<Output = Result<Option<PurchaseHint>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn list_deals<'life0, 'async_trait>(
        &'life0 self,
        _params: ListDealsParams,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn catalog_detail<'life0, 'async_trait>(
        &'life0 self,
        _params: CatalogDetailParams,
    ) -> Pin<Box<dyn Future<Output = Result<Option<CatalogHit>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn revoke_account<'life0, 'async_trait>(
        &'life0 self,
        _params: RevokeAccountParams,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn health<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<HealthOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn diagnose<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Storefront content source (not byte [`Source`](#source)).

Every method takes and returns the typed Cap'n Proto payload structs from
[`crate::generated`](generated.md); absent methods return
[`PluginError::unsupported`](error.md#pluginerrorunsupported).

#### ContentSource::login

```rust
fn login<'life0, 'async_trait>(
    &'life0 self,
    _params: LoginParams,
) -> Pin<Box<dyn Future<Output = Result<LoginResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Password or one-shot OAuth login. The host seals
[`LoginResult::credentials`](generated.md#loginresult) into `encrypted_secrets`.

#### ContentSource::scan

```rust
fn scan<'life0, 'async_trait>(
    &'life0 self,
    _params: ScanParams,
) -> Pin<Box<dyn Future<Output = Result<ScanSummary>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Library scan; the host upserts [`ScanSummary::books`](generated.md#scansummary).

#### ContentSource::fetch_title

```rust
fn fetch_title<'life0, 'async_trait>(
    &'life0 self,
    _params: FetchTitleParams,
) -> Pin<Box<dyn Future<Output = Result<PlainFetch>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Fetch one title into `params.cache_dir` and return plain media paths.

#### ContentSource::list_accounts

```rust
fn list_accounts<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<SourceAccount>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Accounts the guest knows about.

#### ContentSource::login_start

```rust
fn login_start<'life0, 'async_trait>(
    &'life0 self,
    _params: LoginParams,
) -> Pin<Box<dyn Future<Output = Result<LoginStartResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Begin an interactive OAuth login.

#### ContentSource::login_complete

```rust
fn login_complete<'life0, 'async_trait>(
    &'life0 self,
    _params: LoginCompleteParams,
) -> Pin<Box<dyn Future<Output = Result<LoginResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Finish an interactive OAuth login started by [`Self::login_start`](#contentsourcelogin_start).

#### ContentSource::search_catalog

```rust
fn search_catalog<'life0, 'async_trait>(
    &'life0 self,
    _params: SearchCatalogParams,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Free-text storefront catalog search.

#### ContentSource::expand_candidates

```rust
fn expand_candidates<'life0, 'async_trait>(
    &'life0 self,
    _params: ExpandCandidatesParams,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Related-title expansion from a seed title.

#### ContentSource::purchase_hint

```rust
fn purchase_hint<'life0, 'async_trait>(
    &'life0 self,
    _params: PurchaseHintParams,
) -> Pin<Box<dyn Future<Output = Result<Option<PurchaseHint>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Purchase link / price hint; `Ok(None)` when the title is unknown.

#### ContentSource::list_deals

```rust
fn list_deals<'life0, 'async_trait>(
    &'life0 self,
    _params: ListDealsParams,
) -> Pin<Box<dyn Future<Output = Result<Vec<CatalogHit>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Current storefront deals.

#### ContentSource::catalog_detail

```rust
fn catalog_detail<'life0, 'async_trait>(
    &'life0 self,
    _params: CatalogDetailParams,
) -> Pin<Box<dyn Future<Output = Result<Option<CatalogHit>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Full catalog record for one product; `Ok(None)` when unknown.

#### ContentSource::revoke_account

```rust
fn revoke_account<'life0, 'async_trait>(
    &'life0 self,
    _params: RevokeAccountParams,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Undo store-side state for one account before the host deletes its
stored credentials, such as deregistering the device registered at
login.

The default does nothing, which is right for stores that keep no
per-login state.

##### Errors

Returns an error when the store rejects the sign-out. The host logs it
and revokes the account anyway.

#### ContentSource::health

```rust
fn health<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<HealthOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Storefront health.

#### ContentSource::diagnose

```rust
fn diagnose<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Operator-facing diagnostic lines.

Implemented by: `ContentSourceClient`

### Database

```rust
pub trait Database {
    fn open_session<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Box<dyn AdapterDatabaseSession>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn drop_unit<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _unit_ref: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn host_session(&self) -> Option<Box<dyn HostAdapterDatabaseSession>>;
}
```

Database factory. Sessions cannot survive suspension.

#### Database::open_session

```rust
fn open_session<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Box<dyn AdapterDatabaseSession>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Opens an invocation-scoped adapter session.

#### Database::drop_unit

```rust
fn drop_unit<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _unit_ref: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Physically deletes one provisioned binding unit.

`unit_ref` is the adapter-native identity recorded in
`plugin_databases` (SQLite file path, PostgreSQL database name, D1
database name, or a third-party instance id). Missing units are
success so the host can remove the registry row. The default is
unsupported: adapters that cannot prove deletion must fail closed.

#### Database::host_session

```rust
fn host_session(&self) -> Option<Box<dyn HostAdapterDatabaseSession>>
```

Host-private interactive-transaction view of the adapter connection.

First-party adapters override this. The default advertises no host
machinery; hosts then fall back to the public typed `execute` plane.

Implemented by: `DatabaseClient`

### Destination

```rust
pub trait Destination {
    fn head<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Option<ObjectMetadata>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn list<'life0, 'async_trait>(
        &'life0 self,
        options: ListOptions,
    ) -> Pin<Box<dyn Future<Output = Result<ListPage>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn get<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        range: Option<ByteRange>,
    ) -> Pin<Box<dyn Future<Output = Result<ReadResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn put<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
        body: Pin<Box<dyn AsyncRead + Send>>,
        options: WriteOptions,
    ) -> Pin<Box<dyn Future<Output = Result<PutResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn copy<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        from: &'life1 str,
        to: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<CopyResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn delete<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn commit<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        _key: &'life1 str,
        _commit_token: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<PutResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
    fn abort_stage<'life0, 'life1, 'life2, 'async_trait>(
        &'life0 self,
        _key: &'life1 str,
        _commit_token: &'life2 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,
        'life2: 'async_trait,;
}
```

Destination capability (storage).

Cap'n Proto stubs are `!Send`; call these traits from a `LocalSet`.

#### Destination::head

```rust
fn head<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Option<ObjectMetadata>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Metadata without a body; `Ok(None)` when the key is missing.

#### Destination::list

```rust
fn list<'life0, 'async_trait>(
    &'life0 self,
    options: ListOptions,
) -> Pin<Box<dyn Future<Output = Result<ListPage>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

One page of keys under `options.prefix`.

#### Destination::get

```rust
fn get<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    range: Option<ByteRange>,
) -> Pin<Box<dyn Future<Output = Result<ReadResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Streamed read. The body is a transferred stream, not a scalar.

#### Destination::put

```rust
fn put<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
    body: Pin<Box<dyn AsyncRead + Send>>,
    options: WriteOptions,
) -> Pin<Box<dyn Future<Output = Result<PutResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Streamed write. `body` ownership is transferred to the destination.

When [`WriteOptions::stage_only`](#writeoptions) is true, bytes stream into
destination-managed temporary/multipart storage and are not published
until [`Self::commit`](#destinationcommit). Hosts, adapters, brokers, and guests must not
spool the complete object locally.

#### Destination::copy

```rust
fn copy<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    from: &'life1 str,
    to: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<CopyResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Server-side copy when the backend supports it.

#### Destination::delete

```rust
fn delete<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Delete a key (no-op if missing).

#### Destination::commit

```rust
fn commit<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    _key: &'life1 str,
    _commit_token: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<PutResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Finalize a destination-side staged object using `commit_token`.

#### Destination::abort_stage

```rust
fn abort_stage<'life0, 'life1, 'life2, 'async_trait>(
    &'life0 self,
    _key: &'life1 str,
    _commit_token: &'life2 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
    'life2: 'async_trait,
```

Abort a destination-side staged object.

Implemented by: `DestinationClient`

### EventConsumer

```rust
pub trait EventConsumer {
    fn event<'life0, 'async_trait>(
        &'life0 self,
        batch: Vec<DomainEvent>,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<EventResult>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`[[events.consumers]]` handler ([`Entrypoints::event_consumer`](#entrypoints)).

Delivery is at-least-once; consume idempotently on
[`DomainEvent::deduplication_key`](#domainevent).

#### EventConsumer::event

```rust
fn event<'life0, 'async_trait>(
    &'life0 self,
    batch: Vec<DomainEvent>,
) -> Pin<Box<dyn Future<Output = Result<Vec<EventResult>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Handles one ordered batch and returns exactly one [`EventResult`](#eventresult) per
input event, in order. A short reply is a host-side error: the
missing tail is redelivered.

Implemented by: `EventConsumerClient`

### EventPublisher

```rust
pub trait EventPublisher {
    fn publish<'life0, 'async_trait>(
        &'life0 self,
        event: PluginEvent,
    ) -> Pin<Box<dyn Future<Output = Result<PublishOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`EVENTS` binding: host-served outbox publisher ([`Bindings::events`](#bindings)).

#### EventPublisher::publish

```rust
fn publish<'life0, 'async_trait>(
    &'life0 self,
    event: PluginEvent,
) -> Pin<Box<dyn Future<Output = Result<PublishOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Appends `event` to the host outbox. The host stamps `source` (this
plugin id) and the invocation account; `event_type` must be listed in
`[[events.producers]]`.

Implemented by: `EventPublisherClient`

### GuestDatabase

```rust
pub trait GuestDatabase {
    fn execute<'life0, 'async_trait>(
        &'life0 self,
        request: ExecuteRequest,
    ) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn close<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Host-granted SQL transport for job plugin authors (no `capabilities`).

#### GuestDatabase::execute

```rust
fn execute<'life0, 'async_trait>(
    &'life0 self,
    request: ExecuteRequest,
) -> Pin<Box<dyn Future<Output = Result<ExecuteReply>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Host-mediated typed batch (`execute`).

#### GuestDatabase::close

```rust
fn close<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Close the grant.

### JobRunner

```rust
pub trait JobRunner {
    fn job<'life0, 'async_trait>(
        &'life0 self,
        controller: JobController,
    ) -> Pin<Box<dyn Future<Output = Result<JobOutcome>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`[triggers] jobs` handler ([`Entrypoints::job_runner`](#entrypoints)).

#### JobRunner::job

```rust
fn job<'life0, 'async_trait>(
    &'life0 self,
    controller: JobController,
) -> Pin<Box<dyn Future<Output = Result<JobOutcome>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Runs `controller.invocation` using the granted capabilities until
completion, suspension, or cancellation.

Implemented by: `JobRunnerClient`, `StreamCopyHandler`

### KvNamespace

```rust
pub trait KvNamespace {
    fn get<'life0, 'async_trait>(
        &'life0 self,
        key: String,
    ) -> Pin<Box<dyn Future<Output = Result<KvGetOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn put<'life0, 'async_trait>(
        &'life0 self,
        key: String,
        value: Vec<u8>,
        options: KvPutOptions,
    ) -> Pin<Box<dyn Future<Output = Result<KvPutOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn delete<'life0, 'async_trait>(
        &'life0 self,
        key: String,
    ) -> Pin<Box<dyn Future<Output = Result<KvDeleteOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn list<'life0, 'async_trait>(
        &'life0 self,
        options: KvListOptions,
    ) -> Pin<Box<dyn Future<Output = Result<KvListOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Durable key/value namespace: Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names. Values live in the cluster, so an instance
moved to another host reads its own keys.

#### KvNamespace::get

```rust
fn get<'life0, 'async_trait>(
    &'life0 self,
    key: String,
) -> Pin<Box<dyn Future<Output = Result<KvGetOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Reads one key; `found = false` when absent or expired.

#### KvNamespace::put

```rust
fn put<'life0, 'async_trait>(
    &'life0 self,
    key: String,
    value: Vec<u8>,
    options: KvPutOptions,
) -> Pin<Box<dyn Future<Output = Result<KvPutOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Writes one key, replacing any previous value and metadata.

#### KvNamespace::delete

```rust
fn delete<'life0, 'async_trait>(
    &'life0 self,
    key: String,
) -> Pin<Box<dyn Future<Output = Result<KvDeleteOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Deletes one key; a missing key is not an error.

#### KvNamespace::list

```rust
fn list<'life0, 'async_trait>(
    &'life0 self,
    options: KvListOptions,
) -> Pin<Box<dyn Future<Output = Result<KvListOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Lists keys in lexical order, one page at a time.

Implemented by: `KvNamespaceClient`

### Oidc

```rust
pub trait Oidc {
    fn clients<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<OidcClientTemplate>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn authenticate_user<'life0, 'async_trait>(
        &'life0 self,
        _params: AuthenticateUserParams,
    ) -> Pin<Box<dyn Future<Output = Result<ExternalUser>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

#### Oidc::clients

```rust
fn clients<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<OidcClientTemplate>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Plugin-provided OIDC authorization-server client templates. Empty when
the guest only verifies credentials.

#### Oidc::authenticate_user

```rust
fn authenticate_user<'life0, 'async_trait>(
    &'life0 self,
    _params: AuthenticateUserParams,
) -> Pin<Box<dyn Future<Output = Result<ExternalUser>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Verify remote credentials on behalf of the host.

Implemented by: `OidcClient`

### PluginCli

```rust
pub trait PluginCli {
    fn describe<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<CliSchema>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn invoke<'life0, 'async_trait>(
        &'life0 self,
        _params: CliInvokeParams,
    ) -> Pin<Box<dyn Future<Output = Result<CliInvokeResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

#### PluginCli::describe

```rust
fn describe<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<CliSchema>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Declared CLI surface. Empty when the guest exposes no commands.

#### PluginCli::invoke

```rust
fn invoke<'life0, 'async_trait>(
    &'life0 self,
    _params: CliInvokeParams,
) -> Pin<Box<dyn Future<Output = Result<CliInvokeResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Runs one plugin CLI command.

Implemented by: `PluginCliClient`

### PluginWorker

```rust
pub trait PluginWorker: 'static {
    fn describe<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<PluginDescribe>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn open<'life0, 'async_trait>(
        &'life0 self,
        invocation: Invocation,
        bindings: Bindings,
    ) -> Pin<Box<dyn Future<Output = Result<Entrypoints>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn database_migrations<'life0, 'life1, 'async_trait>(
        &'life0 self,
        _binding: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<PluginMigration>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
    fn shutdown<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Root `PluginWorker` capability (`describe` / `open` / `shutdown`).

`describe()` advertises typed capabilities; the verified install receipt plus the
operator grant is the host allowlist. `open()` returns the exported
[`Entrypoints`](#entrypoints) for one invocation.

#### PluginWorker::describe

```rust
fn describe<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<PluginDescribe>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Advertises identity, features, and scalar limits.

#### PluginWorker::open

```rust
fn open<'life0, 'async_trait>(
    &'life0 self,
    invocation: Invocation,
    bindings: Bindings,
) -> Pin<Box<dyn Future<Output = Result<Entrypoints>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Opens the exported entrypoints for `invocation` with the granted
`bindings`.

#### PluginWorker::database_migrations

```rust
fn database_migrations<'life0, 'life1, 'async_trait>(
    &'life0 self,
    _binding: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<Vec<PluginMigration>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Complete ordered plugin-owned migration sequence for one named binding.

Called while the host provisions `[[databases]]`, before the binding
session is handed out on [`Bindings::databases`](#bindings). Empty means the
binding has no plugin-owned migrations. `id` values are opaque
plugin-chosen identities; registration order is the forward sequence.

#### PluginWorker::shutdown

```rust
fn shutdown<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Releases guest resources.

### ProgressSink

```rust
pub trait ProgressSink {
    fn report<'life0, 'life1, 'async_trait>(
        &'life0 self,
        percent: f32,
        message: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

Progress reports for a job invocation (never carries media).

#### ProgressSink::report

```rust
fn report<'life0, 'life1, 'async_trait>(
    &'life0 self,
    percent: f32,
    message: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Reports `percent` in `0..=100` and an operator-facing `message`.

### QueueConsumer

```rust
pub trait QueueConsumer {
    fn queue<'life0, 'async_trait>(
        &'life0 self,
        batch: QueueBatch,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<QueueResult>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`[[queues.consumers]]` handler ([`Entrypoints::queue_consumer`](#entrypoints)).

Delivery is at-least-once; consume idempotently on
[`QueueMessage::id`](generated.md#queuemessage).

#### QueueConsumer::queue

```rust
fn queue<'life0, 'async_trait>(
    &'life0 self,
    batch: QueueBatch,
) -> Pin<Box<dyn Future<Output = Result<Vec<QueueResult>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Handles one ordered batch from one queue and returns exactly one
[`QueueResult`](generated.md#queueresult) per message, in order. A short
reply is a host-side error: the missing tail is redelivered.

Implemented by: `QueueConsumerClient`

### QueueProducer

```rust
pub trait QueueProducer {
    fn send<'life0, 'async_trait>(
        &'life0 self,
        message: QueueSendMessage,
    ) -> Pin<Box<dyn Future<Output = Result<QueueSendOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn send_batch<'life0, 'async_trait>(
        &'life0 self,
        messages: Vec<QueueSendMessage>,
        delay_seconds: u32,
    ) -> Pin<Box<dyn Future<Output = Result<QueueSendOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names ([`Bindings::queues`](#bindings)).

#### QueueProducer::send

```rust
fn send<'life0, 'async_trait>(
    &'life0 self,
    message: QueueSendMessage,
) -> Pin<Box<dyn Future<Output = Result<QueueSendOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Sends one message; the host stamps `source` and assigns the id.

#### QueueProducer::send_batch

```rust
fn send_batch<'life0, 'async_trait>(
    &'life0 self,
    messages: Vec<QueueSendMessage>,
    delay_seconds: u32,
) -> Pin<Box<dyn Future<Output = Result<QueueSendOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Sends up to `MAX_LIST_PAGE` messages in order. `delay_seconds` applies
to every message whose own delay is zero.

Implemented by: `QueueProducerClient`

### RemoteLibrary

```rust
pub trait RemoteLibrary {
    fn health<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<HealthOk>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn start<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn stop<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn diagnose<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn scan_library<'life0, 'async_trait>(
        &'life0 self,
        _params: ScanLibraryParams,
    ) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn sync_listening<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<ListeningProgress>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
    fn poll_events<'life0, 'async_trait>(
        &'life0 self,
    ) -> Pin<Box<dyn Future<Output = Result<Vec<ExternalUser>>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,;
}
```

`remoteLibrary` entrypoint: long-running remote-library lifecycle.

Event delivery is [`EventConsumer`](#eventconsumer); credential verification is [`Oidc`](#oidc).

#### RemoteLibrary::health

```rust
fn health<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<HealthOk>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Liveness.

#### RemoteLibrary::start

```rust
fn start<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Start long-running work after the host has granted bindings.

#### RemoteLibrary::stop

```rust
fn stop<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Stop long-running work.

#### RemoteLibrary::diagnose

```rust
fn diagnose<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<String>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Operator-facing diagnostic lines.

#### RemoteLibrary::scan_library

```rust
fn scan_library<'life0, 'async_trait>(
    &'life0 self,
    _params: ScanLibraryParams,
) -> Pin<Box<dyn Future<Output = Result<()>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Re-sync the remote library.

#### RemoteLibrary::sync_listening

```rust
fn sync_listening<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<ListeningProgress>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Push / pull listening progress; the host upserts the rows.

#### RemoteLibrary::poll_events

```rust
fn poll_events<'life0, 'async_trait>(
    &'life0 self,
) -> Pin<Box<dyn Future<Output = Result<Vec<ExternalUser>>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
```

Drain external users observed since the last poll.

Implemented by: `RemoteLibraryClient`

### Source

```rust
pub trait Source {
    fn open<'life0, 'life1, 'async_trait>(
        &'life0 self,
        key: &'life1 str,
    ) -> Pin<Box<dyn Future<Output = Result<ReadResult>> + 'async_trait>>
    where
        Self: 'async_trait,
        'life0: 'async_trait,
        'life1: 'async_trait,;
}
```

Source capability that can open a named object as a stream.

#### Source::open

```rust
fn open<'life0, 'life1, 'async_trait>(
    &'life0 self,
    key: &'life1 str,
) -> Pin<Box<dyn Future<Output = Result<ReadResult>> + 'async_trait>>
where
    Self: 'async_trait,
    'life0: 'async_trait,
    'life1: 'async_trait,
```

Opens `key` for streamed reading.

Implemented by: `SourceClient`

## Structs

### GuestReceiptPersist

```rust
pub struct GuestReceiptPersist {
    pub guest_statement_len: u32,
    pub guest_request_hash: String,
}
```

Host-stamped hint for adapters to persist guest replay payload before COMMIT.

**Fields**

- `guest_statement_len`: Guest statement count inside the receipt wrap (excluding prune/select).
- `guest_request_hash`: Guest `requestHash` compared on replay.

#### GuestReceiptPersist::is_absent

```rust
pub fn is_absent(&self) -> bool
```

True when the host did not stamp a guest-receipt finalize hint.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AdapterExecuteRequest

```rust
pub struct AdapterExecuteRequest {
    pub request: ExecuteRequest,
    pub guest_receipt: GuestReceiptPersist,
    pub proofs: Vec<ResolvedStatement>,
    pub isolation: IsolationReq,
}
```

Adapter execute payload: canonical SQL, structured proofs, optional receipt.

This is the trust boundary. Transport and adapter execution must call
[`Self::require_proofs`](#adapterexecuterequestrequire_proofs) and must not regenerate, repair, or substitute
proofs.

**Fields**

- `request`: Canonical execute payload (no proofs).
- `guest_receipt`: Host-stamped finalize hint for guest replay.
- `proofs`: One resolved proof per statement, bound to that statement's SQL.
- `isolation`: Isolation the adapter must realize.

#### AdapterExecuteRequest::new

```rust
pub fn new(request: ExecuteRequest, guest_receipt: GuestReceiptPersist) -> Self
```

Builds an adapter execute request without proofs.

Prefer [`UnresolvedExecuteRequest::canonicalize`](#unresolvedexecuterequestcanonicalize) plus
[`CanonicalExecuteRequest::bind_proofs`](#canonicalexecuterequestbind_proofs). This constructor exists for
staged host assembly; [`Self::require_proofs`](#adapterexecuterequestrequire_proofs) must still succeed before
the request crosses the adapter boundary.

#### AdapterExecuteRequest::with_proofs

```rust
pub fn with_proofs(self, proofs: Vec<ResolvedStatement>) -> Self
```

Stamps 1:1 proofs (including receipt wrappers).

#### AdapterExecuteRequest::with_isolation

```rust
pub fn with_isolation(self, isolation: IsolationReq) -> Self
```

Sets isolation (default [`IsolationReq::AtomicBatch`](db_execute.md#isolationreq)).

#### AdapterExecuteRequest::require_proofs

```rust
pub fn require_proofs(&self) -> Result<()>
```

Fail closed unless proofs are 1:1 and hash-bound to each statement.

##### Errors

Returns when the proof count differs or a proof is not bound to its SQL.

Implements: `Clone`, `Debug`

### CanonicalExecuteRequest

```rust
pub struct CanonicalExecuteRequest {
    pub request: ExecuteRequest,
    pub guest_receipt: GuestReceiptPersist,
    pub isolation: IsolationReq,
}
```

Host-desugared canonical SQL. Proofs are not yet bound.

The SQL strings here are the exact text that must cross the adapter
boundary and that proofs must hash.

**Fields**

- `request`: Already-desugared canonical statements (`?` placeholders).
- `guest_receipt`: Host-stamped finalize hint for guest replay.
- `isolation`: Isolation the adapter must realize.

#### CanonicalExecuteRequest::from_desugared

```rust
pub fn from_desugared(request: ExecuteRequest) -> Self
```

Wraps already-desugared SQL (for example after [`crate::desugar_execute_request`](sql_desugar.md#desugar_execute_request)).

#### CanonicalExecuteRequest::with_guest_receipt

```rust
pub fn with_guest_receipt(self, guest_receipt: GuestReceiptPersist) -> Self
```

Host-stamped finalize hint for guest replay.

#### CanonicalExecuteRequest::with_isolation

```rust
pub fn with_isolation(self, isolation: IsolationReq) -> Self
```

Isolation the adapter must realize.

#### CanonicalExecuteRequest::bind_proofs

```rust
pub fn bind_proofs(self, proofs: Vec<ResolvedStatement>) -> Result<AdapterExecuteRequest>
```

Binds 1:1 proofs to the canonical SQL. Fail closed on mismatch.

##### Errors

Returns when the proof count differs or a proof is not bound to its SQL.

Implements: `Clone`, `Debug`

### UnresolvedExecuteRequest

```rust
pub struct UnresolvedExecuteRequest {
    pub request: ExecuteRequest,
    pub guest_receipt: GuestReceiptPersist,
    pub isolation: IsolationReq,
}
```

Source SQL that has not been host-canonicalized.

Construct this from planner output or SeaORM SQL, then
[`Self::canonicalize`](#unresolvedexecuterequestcanonicalize) exactly once before proof generation.

**Fields**

- `request`: Source statements (may still need `NULLS` / `NULLIF` desugar).
- `guest_receipt`: Host-stamped finalize hint for guest replay.
- `isolation`: Isolation the adapter must realize.

#### UnresolvedExecuteRequest::new

```rust
pub fn new(request: ExecuteRequest) -> Self
```

Wraps source SQL with default isolation and no receipt hint.

#### UnresolvedExecuteRequest::with_guest_receipt

```rust
pub fn with_guest_receipt(self, guest_receipt: GuestReceiptPersist) -> Self
```

Host-stamped finalize hint for guest replay.

#### UnresolvedExecuteRequest::with_isolation

```rust
pub fn with_isolation(self, isolation: IsolationReq) -> Self
```

Isolation the adapter must realize.

#### UnresolvedExecuteRequest::canonicalize

```rust
pub fn canonicalize(self) -> CanonicalExecuteRequest
```

Applies Bookclerk semantic desugars exactly once.

Implements: `Clone`, `Debug`

### PluginMigration

```rust
pub struct PluginMigration {
    pub id: String,
    pub operations: Vec<PluginMigrationOp>,
}
```

One plugin-owned migration application in registration order.

**Fields**

- `id`: Opaque plugin-chosen stable identity.
- `operations`: Ordered already-separated BookclerkSQL operations.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### IntegerArithSite

```rust
pub struct IntegerArithSite {
    pub full: SqlSpan,
    pub lhs: SqlSpan,
    pub rhs: SqlSpan,
    pub kind: IntegerArithKind,
}
```

One INTEGER arithmetic expression that must not wrap or error on overflow.

**Fields**

- `full`: Full expression span (`a + b` or `abs(n)`).
- `lhs`: Left operand (or `abs` argument).
- `rhs`: Right operand (`abs` repeats `lhs`).
- `kind`: Operator.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PhysicalAccess

```rust
pub struct PhysicalAccess {
    pub table: String,
    pub column: Option<String>,
}
```

Physical table/column access used for authorization.

**Fields**

- `table`: Folded physical table name.
- `column`: Folded column name when the access is column-specific.
  
  [`PHYSICAL_STAR_COLUMN`](#physical_star_column) (`"*"`) means a projection wildcard on this
  table (`SELECT *`, `alias.*`, `RETURNING *`). `None` is table presence
  only (FROM / INSERT / UPDATE / DELETE / DROP), not a wildcard.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`

### ResolvedAssignment

```rust
pub struct ResolvedAssignment {
    pub table: String,
    pub column: String,
    pub dest: SqlType,
    pub source: SqlType,
}
```

Destination assignment `lhs = rhs` (INSERT, UPDATE, …).

**Fields**

- `table`: Physical table of the destination column.
- `column`: Destination column.
- `dest`: Destination type.
- `source`: Unified source type.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ResolvedStatement

```rust
pub struct ResolvedStatement {
    pub statement_hash: String,
    pub output_columns: Vec<(String, SqlType)>,
    pub physical_accesses: Vec<PhysicalAccess>,
    pub assignments: Vec<ResolvedAssignment>,
    pub text_collate_sites: Vec<TextCollateSite>,
    pub integer_arith_sites: Vec<IntegerArithSite>,
    pub functions: Vec<String>,
    pub schema_action: SchemaAction,
}
```

One resolved typed proof bound to an exact canonical statement.

**Fields**

- `statement_hash`: SHA-256 hex of the exact canonical SQL this proof claims.
- `output_columns`: SELECT / RETURNING / VALUES output columns in order.
- `physical_accesses`: Physical tables/columns referenced (authorization).
- `assignments`: Mutation destination assignments.
- `text_collate_sites`: TEXT expression spans for `COLLATE "C"` (already excluding dest/FROM/AS).
- `integer_arith_sites`: INTEGER overflow sites (`+` `-` `*` `abs`).
- `functions`: Function names invoked in code spans (folded), for authorization.
- `schema_action`: DDL action + fingerprint.

#### ResolvedStatement::bound_empty

```rust
pub fn bound_empty(sql: &str) -> Self
```

Proof for `sql` with empty semantic payloads (wrapper DDL/DML still hash-bound).

#### ResolvedStatement::proves

```rust
pub fn proves(&self, sql: &str) -> bool
```

True when `sql` is the exact statement this proof was built from.

#### ResolvedStatement::validate_for

```rust
pub fn validate_for(&self, sql: &str) -> Result<()>
```

Rejects a hash-correct proof whose TEXT/arithmetic sidecar cannot be applied.

##### Errors

Returns when a span is empty, out of range, not on a UTF-8 boundary,
operands are not contained in `full`, sites partially overlap, or an
arithmetic kind does not match the operator in `sql`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SqlSpan

```rust
pub struct SqlSpan {
    pub start: usize,
    pub end: usize,
}
```

Byte span in the exact canonical SQL string a proof is bound to.

**Fields**

- `start`: Inclusive start byte offset.
- `end`: Exclusive end byte offset.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TextCollateSite

```rust
pub struct TextCollateSite {
    pub span: SqlSpan,
}
```

One TEXT expression that PostgreSQL must collate as `"C"`.

**Fields**

- `span`: Identifier or string-literal span in canonical SQL.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SqlFnCall

```rust
pub struct SqlFnCall {
    pub name: String,
    pub arg_count: usize,
}
```

One helper call found in canonical SQL (code spans only).

**Fields**

- `name`: Folded helper name (`json_object`, `replace`, …).
- `arg_count`: Top-level argument count (`json_object('k', 1)` is 2).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### HostAdapterDatabaseSessionClient

```rust
pub struct HostAdapterDatabaseSessionClient {
    /* private fields */
}
```

Host-side client for `begin` on an open adapter session capability.

#### HostAdapterDatabaseSessionClient::from_session_client

```rust
pub fn from_session_client(client: Client) -> Self
```

Casts a public adapter session capability to the host-private interface.

Implements: `HostAdapterDatabaseSession`

### RpcFeature

```rust
pub struct RpcFeature {
    pub name: &'static str,
}
```

Known RPC feature names.

**Fields**

- `name`: Wire name (`rpc.streams`, …).

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### StreamCopyHandler

```rust
pub struct StreamCopyHandler;
```

Default [`JobRunner`](#jobrunner) for the stream-copy vertical slice.

Implements: `JobRunner`

### StreamCopySpec

```rust
pub struct StreamCopySpec {
    pub from: String,
    pub to: String,
}
```

JSON body for a `stream_copy` [`JobInvocation`](#jobinvocation).

**Fields**

- `from`: Source object key.
- `to`: Destination object key.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScalarLimits

```rust
pub struct ScalarLimits {
    pub max_scalar_bytes: u32,
    pub max_stream_window_bytes: u32,
    pub max_list_page: u32,
}
```

Negotiated numeric limits advertised at [`super::PluginDescribe`](#plugindescribe).

**Fields**

- `max_scalar_bytes`: [`MAX_SCALAR_BYTES`](#max_scalar_bytes) (or a host-clamped lower value).
- `max_stream_window_bytes`: [`MAX_STREAM_WINDOW_BYTES`](#max_stream_window_bytes) (or a host-clamped lower value).
- `max_list_page`: [`MAX_LIST_PAGE`](#max_list_page) (or a host-clamped lower value).

#### ScalarLimits::validate

```rust
pub fn validate(self) -> Result<Self>
```

Rejects zero or above-product-maxima limits.

##### Errors

Returns [`crate::PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when any component is
zero or larger than the product constant.

#### ScalarLimits::intersect

```rust
pub fn intersect(self, other: Self) -> Self
```

Intersection of host-offered and guest-accepted limits (component-wise min).

#### ScalarLimits::clamp_list_limit

```rust
pub fn clamp_list_limit(self, limit: u32) -> u32
```

Clamps a list page `limit` into `(0, max_list_page]`.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Eq`, `From<ScalarLimits>`, `From<ScalarLimitsDto>`, `PartialEq`

### BindingValues

```rust
pub struct BindingValues {
    pub config: ExtensibleConfig,
    pub secrets: ExtensibleConfig,
    pub adapter: DatabaseAdapterConfig,
}
```

Plain-data portion of [`Bindings`]: every granted value that is not a
capability. Hosts build this off the vat thread and attach capabilities
when they call `open`.

**Fields**

- `config`: `CONFIG`: granted plugin settings as `application/json`.
- `secrets`: `SECRETS`: granted secret values as `application/json`; empty payload
  when the manifest declares no `[secrets]`.
- `adapter`: Database-adapter bootstrap for the `databaseAdapter` entrypoint.
  First-party host-managed adapters receive host-private connect params
  in [`Self::config`](#bindingvaluesconfig); third-party adapters receive this typed bootstrap
  (and an empty `config`). `plugin_data_dir` is empty when `config`
  carries host-private params instead.

#### BindingValues::config

```rust
pub fn config(config: ExtensibleConfig) -> Self
```

Bindings carrying only `CONFIG`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Bindings

```rust
pub struct Bindings {
    pub config: ExtensibleConfig,
    pub secrets: ExtensibleConfig,
    pub adapter: DatabaseAdapterConfig,
    pub events: Option<Box<dyn EventPublisher>>,
    pub databases: Vec<(String, Box<dyn GuestDatabase>)>,
    pub cancel: Box<dyn Cancellation>,
    pub storage: Option<Box<dyn Destination>>,
    pub queues: Vec<NamedQueueBinding>,
    pub kv: Vec<NamedKvBinding>,
}
```

Host-granted bindings for one [`PluginWorker::open`](#pluginworkeropen) (`env` in the
Workers idiom). Capability fields are `None` when the manifest does not
declare (or the operator did not grant) the binding.

**Fields**

- `config`: `CONFIG`: granted plugin settings as `application/json`.
- `secrets`: `SECRETS`: granted secret values; empty payload when none.
- `adapter`: Database-adapter bootstrap (see [`BindingValues::adapter`](#bindingvalues)).
- `events`: `EVENTS`: outbox publisher; `None` unless `[[events.producers]]` is
  granted.
- `databases`: Named plugin-owned `[[databases]]` bindings: isolated databases
  separate from the Bookclerk library and from every other plugin.
- `cancel`: Host cancellation for the whole invocation (fence / lease loss).
- `storage`: `WORK_FS`: host-granted object storage for durable plugin files;
  `None` unless `[work_fs]` is granted. Job input/output travel on
  [`JobController`](#jobcontroller), never here.
- `queues`: Named `[[queues.producers]]` bindings; empty when none is granted.
- `kv`: Named `[[kv_namespaces]]` bindings; empty when none is granted.

#### Bindings::from_values

```rust
pub fn from_values(values: BindingValues) -> Self
```

Bindings with only plain values (no capabilities); tests and
in-process hosts.

#### Bindings::take_named_queue

```rust
pub fn take_named_queue(&mut self, name: &str) -> Option<NamedQueueBinding>
```

Takes the named queue producer binding `name`, if granted.

#### Bindings::take_named_kv

```rust
pub fn take_named_kv(&mut self, name: &str) -> Option<NamedKvBinding>
```

Takes the named KV namespace binding `name`, if granted.

#### Bindings::values

```rust
pub fn values(&self) -> BindingValues
```

Plain-data view (`config` / `secrets` / `adapter`).

#### Bindings::take_named_database

```rust
pub fn take_named_database(&mut self, name: &str) -> Option<Box<dyn GuestDatabase>>
```

Takes the named plugin database binding `name`, if granted.

### ByteRange

```rust
pub struct ByteRange {
    pub offset: u64,
    pub length: Option<u64>,
}
```

Inclusive byte range for a streamed read.

**Fields**

- `offset`: Starting offset.
- `length`: Number of bytes; `None` means to end of object.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### Entrypoints

```rust
pub struct Entrypoints {
    pub event_consumer: Option<Box<dyn EventConsumer>>,
    pub job_runner: Option<Box<dyn JobRunner>>,
    pub storefront: Option<Box<dyn ContentSource>>,
    pub storage: Option<Box<dyn Destination>>,
    pub database_adapter: Option<Box<dyn Database>>,
    pub remote_library: Option<Box<dyn RemoteLibrary>>,
    pub cli: Option<Box<dyn PluginCli>>,
    pub oidc: Option<Box<dyn Oidc>>,
    pub queue_consumer: Option<Box<dyn QueueConsumer>>,
}
```

Exported entrypoints returned by [`PluginWorker::open`]: one capability
per `plugin.toml` `entrypoints` entry / trigger, `None` when not exported.

The host refuses an entrypoint the manifest or operator grant did not
allow, so a guest cannot widen its surface here.

**Fields**

- `event_consumer`: `[[events.consumers]]` trigger.
- `job_runner`: `[triggers] jobs` trigger.
- `storefront`: `storefront` entrypoint.
- `storage`: `storage` entrypoint.
- `database_adapter`: `databaseAdapter` entrypoint.
- `remote_library`: `remoteLibrary` entrypoint.
- `cli`: `cli` entrypoint.
- `oidc`: `oidc` entrypoint.
- `queue_consumer`: `[[queues.consumers]]` trigger.

Implements: `Default`

### JobController

```rust
pub struct JobController {
    pub invocation: JobInvocation,
    pub input: Box<dyn Source>,
    pub output: Box<dyn Destination>,
    pub progress: Box<dyn ProgressSink>,
    pub cancel: Box<dyn Cancellation>,
}
```

Everything one [`JobRunner::job`](#jobrunnerjob) invocation may touch.

Named database bindings are not per job: they arrive on
[`Bindings::databases`](#bindings) at [`PluginWorker::open`](#pluginworkeropen).

**Fields**

- `invocation`: Durable command envelope.
- `input`: Input source capability.
- `output`: Output destination capability.
- `progress`: Progress sink (durable job row).
- `cancel`: Cancellation capability (host fence / lease).

### NamedKvBinding

```rust
pub struct NamedKvBinding {
    pub name: String,
    pub namespace: String,
    pub kv: Box<dyn KvNamespace>,
}
```

One granted `[[kv_namespaces]]` binding on [`Bindings::kv`](#bindings).

**Fields**

- `name`: Binding name on `env` (`[[kv_namespaces]] binding`).
- `namespace`: Namespace the binding is scoped to.
- `kv`: Host-served namespace.

### NamedQueueBinding

```rust
pub struct NamedQueueBinding {
    pub name: String,
    pub queue: String,
    pub producer: Box<dyn QueueProducer>,
}
```

One granted `[[queues.producers]]` binding on [`Bindings::queues`](#bindings).

**Fields**

- `name`: Binding name on `env` (`[[queues.producers]] binding`).
- `queue`: Queue the binding sends to.
- `producer`: Host-served producer.

### NeverCancel

```rust
pub struct NeverCancel;
```

Cancellation that never fires (tests / hosts without a fence).

Implements: `Cancellation`

### ReadResult

```rust
pub struct ReadResult {
    pub meta: ObjectMetadata,
    pub body: Pin<Box<dyn AsyncRead + Send>>,
}
```

Streamed read result. `body` ownership is transferred to the caller.

**Fields**

- `meta`: Object metadata (size, type, checksums).
- `body`: Byte stream; drop/cancel aborts the read.

### AdapterSessionHandle

```rust
pub struct AdapterSessionHandle {
    pub session: Box<dyn AdapterDatabaseSession>,
    pub host: HostAdapterDatabaseSessionClient,
}
```

Public adapter session plus host-private interactive transaction client.

**Fields**

- `session`: Typed `capabilities` / `execute` / `close`.
- `host`: Host-only `begin` (capability cast on the same session object).

### ContentSourceClient

```rust
pub struct ContentSourceClient {
    /* private fields */
}
```

Cap'n Proto client for [`ContentSource`](#contentsource).

#### ContentSourceClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `ContentSource` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `ContentSource`

### DatabaseClient

```rust
pub struct DatabaseClient {
    /* private fields */
}
```

Cap'n Proto client for [`Database`](#database).

#### DatabaseClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `Database` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

#### DatabaseClient::open_session_handle

```rust
pub async fn open_session_handle(&self) -> Result<AdapterSessionHandle>
```

Opens a session and host-private transaction client on one capability.

##### Errors

Returns when `openSession` fails on the guest.

Implements: `Clone`, `Database`

### DestinationClient

```rust
pub struct DestinationClient {
    /* private fields */
}
```

Host-side [`Destination`](#destination) over a capnp destination stub.

#### DestinationClient::new

```rust
pub fn new(client: Client, window: u32) -> Self
```

Wraps a capnp destination client.

#### DestinationClient::with_max_list_page

```rust
pub fn with_max_list_page(self, max_list_page: u32) -> Self
```

Applies negotiated list-page cardinality.

Implements: `Clone`, `Destination`

### DestinationServer

```rust
pub struct DestinationServer {
    /* private fields */
}
```

Cap'n Proto server wrapping a [`Destination`](#destination) trait object.

#### DestinationServer::new

```rust
pub fn new(inner: Arc<dyn Destination>, window: u32) -> Self
```

Serves `inner` with the given stream window.

Implements: `Server`

### EventConsumerClient

```rust
pub struct EventConsumerClient {
    /* private fields */
}
```

Cap'n Proto client for [`EventConsumer`](#eventconsumer) (host → guest delivery).

#### EventConsumerClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `EventConsumer` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `EventConsumer`

### EventPublisherClient

```rust
pub struct EventPublisherClient {
    /* private fields */
}
```

Guest-side client for the host `EVENTS` binding ([`EventPublisher`](#eventpublisher)).

Implements: `Clone`, `EventPublisher`

### HostBindings

```rust
pub struct HostBindings {
    pub values: BindingValues,
    pub events: Option<Arc<dyn EventPublisher>>,
    pub databases: Vec<(String, Arc<dyn GuestDatabase>)>,
    pub cancel: Arc<dyn Cancellation>,
    pub storage: Option<Arc<dyn Destination>>,
    pub queues: Vec<(String, String, Arc<dyn QueueProducer>)>,
    pub kv: Vec<(String, String, Arc<dyn KvNamespace>)>,
}
```

Host-side [`Bindings`]: plain values plus host-served capabilities the
guest receives on `open`.

**Fields**

- `values`: `CONFIG` / `SECRETS` / adapter bootstrap.
- `events`: `EVENTS` publisher; `None` unless `[[events.producers]]` is granted.
- `databases`: Named plugin-owned `[[databases]]` sessions.
- `cancel`: Invocation-wide cancellation (fence / lease loss).
- `storage`: `WORK_FS` object storage; `None` unless `[work_fs]` is granted.
- `queues`: Named `[[queues.producers]]` producers as `(binding, queue, producer)`.
- `kv`: Named `[[kv_namespaces]]` namespaces as `(binding, namespace, kv)`.

#### HostBindings::from_values

```rust
pub fn from_values(values: BindingValues) -> Self
```

Bindings with only plain values and no cancellation source.

Implements: `Default`

### JobRunnerClient

```rust
pub struct JobRunnerClient {
    /* private fields */
}
```

Cap'n Proto client for [`JobRunner`](#jobrunner) (host → guest job dispatch).

#### JobRunnerClient::new

```rust
pub fn new(client: Client, window: u32) -> Self
```

Wraps a `JobRunner` capability client with the negotiated stream window.

#### JobRunnerClient::job

```rust
pub async fn job(
    &self,
    invocation: &JobInvocation,
    input: Arc<dyn Source>,
    output: Arc<dyn Destination>,
    progress: Arc<dyn ProgressSink>,
    cancel: Arc<dyn Cancellation>,
) -> Result<JobOutcome>
```

Runs one job with host-served input / output / progress / cancel
capabilities.

##### Errors

Returns a plugin error when the runner fails.

#### JobRunnerClient::job_with_capabilities

```rust
pub async fn job_with_capabilities(
    &self,
    invocation: &JobInvocation,
    input: Client,
    output: Client,
    progress: Client,
    cancel: Client,
) -> Result<JobOutcome>
```

Runs one job with caller-built `JobController` capability clients.

Transports that do not host a local vat (the bookclerk-workerd
`/invoke` hook, where each capability is a grant-token descriptor)
build the four clients themselves and reuse the typed envelope here.

##### Errors

Returns a plugin error when the runner fails.

Implements: `Clone`, `JobRunner`

### KvNamespaceClient

```rust
pub struct KvNamespaceClient {
    /* private fields */
}
```

Guest-side client for a host KV namespace binding ([`KvNamespace`](#kvnamespace)).

#### KvNamespaceClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `KvNamespace` capability client.

Implements: `Clone`, `KvNamespace`

### OidcClient

```rust
pub struct OidcClient {
    /* private fields */
}
```

Cap'n Proto client for [`Oidc`](#oidc).

#### OidcClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `Oidc` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `Oidc`

### OpenedEntrypoints

```rust
pub struct OpenedEntrypoints {
    pub event_consumer: Option<EventConsumerClient>,
    pub job_runner: Option<JobRunnerClient>,
    pub storefront: Option<ContentSourceClient>,
    pub storage: Option<DestinationClient>,
    pub database_adapter: Option<DatabaseClient>,
    pub remote_library: Option<RemoteLibraryClient>,
    pub cli: Option<PluginCliClient>,
    pub oidc: Option<OidcClient>,
    pub queue_consumer: Option<QueueConsumerClient>,
}
```

Typed clients for the entrypoints a guest exported from `open`. `None`
means the guest did not export that entrypoint.

**Fields**

- `event_consumer`: `[[events.consumers]]` trigger.
- `job_runner`: `[triggers] jobs` trigger.
- `storefront`: `storefront` entrypoint.
- `storage`: `storage` entrypoint.
- `database_adapter`: `databaseAdapter` entrypoint.
- `remote_library`: `remoteLibrary` entrypoint.
- `cli`: `cli` entrypoint.
- `oidc`: `oidc` entrypoint.
- `queue_consumer`: `[[queues.consumers]]` trigger.

Implements: `Default`

### PluginCliClient

```rust
pub struct PluginCliClient {
    /* private fields */
}
```

Cap'n Proto client for [`PluginCli`](#plugincli).

#### PluginCliClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `PluginCli` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `PluginCli`

### PluginClient

```rust
pub struct PluginClient {
    pub window: u32,
    pub limits: ScalarLimits,
    /* private fields */
}
```

Host bootstrap client for a plugin vat.

**Fields**

- `window`: Negotiated stream window.
- `limits`: Negotiated scalar / list limits.

#### PluginClient::new

```rust
pub fn new(client: Client, window: u32) -> Self
```

Wraps a bootstrap client.

#### PluginClient::with_limits

```rust
pub fn with_limits(self, limits: ScalarLimits) -> Self
```

Applies negotiated limits (stream window + list page).

#### PluginClient::describe

```rust
pub async fn describe(&self) -> Result<PluginDescribe>
```

Calls `describe`.

##### Errors

Returns a plugin error when the RPC fails or `apiVersion` is not
[`crate::limits::PRODUCT_API_VERSION`](#product_api_version).

#### PluginClient::open

```rust
pub async fn open(
    &self,
    invocation: &Invocation,
    bindings: HostBindings,
) -> Result<OpenedEntrypoints>
```

Opens the exported entrypoints for `invocation` with host-granted
`bindings`.

##### Errors

Returns a plugin error when the RPC fails or the guest refuses the open.

#### PluginClient::database_migrations

```rust
pub async fn database_migrations(&self, binding: &str) -> Result<Vec<PluginMigration>>
```

Complete ordered plugin-owned migration sequence for one named binding.

##### Errors

Returns a plugin error when the RPC fails.

#### PluginClient::shutdown

```rust
pub async fn shutdown(&self) -> Result<()>
```

Asks the guest to release its resources (`PluginWorker.shutdown`).

##### Errors

Returns a plugin error when the RPC fails or the guest reports one.

Implements: `Clone`

### PluginServer

```rust
pub struct PluginServer {
    /* private fields */
}
```

Cap'n Proto server wrapping [`PluginWorker`](#pluginworker).

#### PluginServer::new

```rust
pub fn new(inner: Arc<dyn PluginWorker>, window: u32) -> Self
```

Serves `inner`.

Implements: `Server`

### QueueConsumerClient

```rust
pub struct QueueConsumerClient {
    /* private fields */
}
```

Cap'n Proto client for [`QueueConsumer`](#queueconsumer) (host → guest delivery).

#### QueueConsumerClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `QueueConsumer` capability client (any transport that yields
one, including the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `QueueConsumer`

### QueueProducerClient

```rust
pub struct QueueProducerClient {
    /* private fields */
}
```

Guest-side client for a host queue producer binding ([`QueueProducer`](#queueproducer)).

#### QueueProducerClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `QueueProducer` capability client.

Implements: `Clone`, `QueueProducer`

### RemoteLibraryClient

```rust
pub struct RemoteLibraryClient {
    /* private fields */
}
```

Cap'n Proto client for [`RemoteLibrary`](#remotelibrary).

#### RemoteLibraryClient::new

```rust
pub fn new(client: Client) -> Self
```

Wraps a `RemoteLibrary` capability client (any transport that yields one, including
the bookclerk-workerd `/invoke` hook).

Implements: `Clone`, `RemoteLibrary`

### SourceClient

```rust
pub struct SourceClient {
    /* private fields */
}
```

Host-side [`Source`](#source) over a capnp source stub.

#### SourceClient::new

```rust
pub fn new(client: Client, window: u32) -> Self
```

Wraps a capnp source client.

Implements: `Clone`, `Source`

### SourceServer

```rust
pub struct SourceServer {
    /* private fields */
}
```

Cap'n Proto server wrapping a [`Source`](#source).

#### SourceServer::new

```rust
pub fn new(inner: Arc<dyn Source>, window: u32) -> Self
```

Serves `inner`.

Implements: `Server`

### CopyResult

```rust
pub struct CopyResult {
    pub bytes_copied: u64,
}
```

Result of a server-side copy.

**Fields**

- `bytes_copied`: Bytes copied when known.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DomainEvent

```rust
pub struct DomainEvent {
    pub event_id: String,
    pub event_type: String,
    pub schema_version: u32,
    pub occurred_at_unix_ms: u64,
    pub account_id: String,
    pub source: String,
    pub correlation_id: String,
    pub causation_id: String,
    pub deduplication_key: String,
    pub delivery_attempt: u32,
    pub payload: Vec<u8>,
    pub checkpoint_json: String,
    pub checkpoint_schema_version: u32,
    pub invocation_sequence: u32,
    pub resume_pending: bool,
}
```

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

**Fields**

- `event_id`: Unique event id.
- `event_type`: Event type name.
- `schema_version`: Payload schema version.
- `occurred_at_unix_ms`: UTC unix milliseconds.
- `account_id`: Tenant / account id.
- `source`: Producer plugin id; empty when unknown.
- `correlation_id`: Trace correlation id.
- `causation_id`: Prior event or invocation that caused this one.
- `deduplication_key`: Idempotent consume key.
- `delivery_attempt`: 1-based delivery attempt.
- `payload`: Bounded payload.
- `checkpoint_json`: Checkpoint JSON from a prior [`EventResult::Suspended`](#eventresult) (≤ [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes)).
- `checkpoint_schema_version`: Schema version of [`Self::checkpoint_json`](#domainevent).
- `invocation_sequence`: Resume ordinal copied from the delivery row.
- `resume_pending`: True when this invocation continues a prior `suspended` result.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ExtensibleConfig

```rust
pub struct ExtensibleConfig {
    pub schema_version: u32,
    pub media_type: String,
    pub payload: Vec<u8>,
}
```

Versioned plugin-specific config (not a substitute for typed ABI fields).

**Fields**

- `schema_version`: Payload schema version.
- `media_type`: IANA media type or `application/json`.
- `payload`: Bounded payload bytes.

#### ExtensibleConfig::json

```rust
pub fn json(value: &Value) -> Self
```

Wraps a JSON document as an `application/json` payload (schema version 1).

#### ExtensibleConfig::json_from

```rust
pub fn json_from<T: Serialize>(value: &T) -> Result<Self>
```

Serializes `value` as an `application/json` payload (schema version 1).

##### Errors

Returns [`crate::PluginErrorCode::Internal`](error.md#pluginerrorcode) when `value` cannot be
serialized.

#### ExtensibleConfig::is_empty

```rust
pub fn is_empty(&self) -> bool
```

True when no payload was supplied.

#### ExtensibleConfig::json_value

```rust
pub fn json_value(&self) -> Result<Value>
```

Decodes the payload as a JSON document; an empty payload is `{}`.

##### Errors

Returns [`crate::PluginErrorCode::InvalidParams`](error.md#pluginerrorcode) when the media type is
not JSON or the bytes are not valid JSON.

#### ExtensibleConfig::json_into

```rust
pub fn json_into<T: DeserializeOwned>(&self) -> Result<T>
```

Decodes the JSON payload into `T`; an empty payload decodes `{}`.

##### Errors

Same as [`Self::json_value`](#extensibleconfigjson_value), plus a deserialization failure into `T`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### HealthOk

```rust
pub struct HealthOk {
    pub ok: bool,
    pub detail: String,
}
```

Health payload.

**Fields**

- `ok`: True when the role is ready.
- `detail`: Operator-facing detail.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobCheckpoint

```rust
pub struct JobCheckpoint {
    pub schema_version: u32,
    pub json: String,
}
```

Bounded, versioned checkpoint attached to an invocation or suspension.

**Fields**

- `schema_version`: Checkpoint payload schema version. Unknown versions fail closed.
- `json`: JSON checkpoint bytes (must be ≤ [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes)).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobInvocation

```rust
pub struct JobInvocation {
    pub payload_schema_version: u32,
    pub invocation_id: String,
    pub command_type: String,
    pub payload_json: String,
    pub idempotency_key: String,
    pub attempt: u32,
    pub correlation_id: String,
    pub causation_id: Option<String>,
    pub deadline_unix_ms: u64,
    pub checkpoint: Option<JobCheckpoint>,
    pub invocation_sequence: u32,
    pub step_id: Option<String>,
}
```

Versioned durable command envelope (not a domain event).

Idempotency keys are scoped to `(account, plugin, commandType)` until a
terminal fenced outcome is committed. They are not reusable across accounts.
`deadline_unix_ms` is a guest hint; the host fence/lease is authoritative
and must not be outlived (clock skew across VPS nodes).
The envelope is not independently persisted as opaque bytes across ABI
majors; command `payload_schema_version` and checkpoint schema versions
version the stored pieces.

**Fields**

- `payload_schema_version`: Command payload schema version for [`Self::command_type`](#jobinvocation).
- `invocation_id`: Unique id for this attempt envelope.
- `command_type`: Command type (`stream_copy`, …).
- `payload_json`: JSON payload for that command type.
- `idempotency_key`: Idempotency key (see struct docs for scope).
- `attempt`: 1-based attempt counter.
- `correlation_id`: Correlation id for traces.
- `causation_id`: Optional causation id (prior invocation that produced this one).
- `deadline_unix_ms`: Guest-visible UTC deadline as unix milliseconds.
- `checkpoint`: Bounded, versioned checkpoint from a prior suspension.
- `invocation_sequence`: Resume ordinal; distinct from failure [`Self::attempt`](#jobinvocation).
- `step_id`: Deterministic step identifier within this invocation.

#### JobInvocation::stream_copy

```rust
pub fn stream_copy(job_id: impl Into<String>, payload_json: impl Into<String>) -> Self
```

Builds a v1 `stream_copy` invocation with a far-future deadline.

Tests and accountless helpers use this. Production copy jobs should call
[`Self::stream_copy_from_lease`](#jobinvocationstream_copy_from_lease) so attempt, deadline, and idempotency
come from the claimed fence.

#### JobInvocation::stream_copy_from_lease

```rust
pub fn stream_copy_from_lease(
    lease: JobInvocationLease,
    payload_json: impl Into<String>,
) -> Self
```

Builds a `stream_copy` envelope from a claimed job lease.

`invocation_id` is unique per attempt (`job_id:attempt:generation`).
`idempotency_key` is the durable dedup key and stays stable across
reclaim. `invocation_sequence` is the resume ordinal from the lease
(distinct from failure `attempt`). `deadline_unix_ms` is the lease
expiry as unix milliseconds.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobInvocationLease

```rust
pub struct JobInvocationLease {
    pub job_id: String,
    pub attempt: u32,
    pub generation: i64,
    pub dedup_key: String,
    pub deadline_unix_ms: u64,
    pub checkpoint: Option<JobCheckpoint>,
    pub invocation_sequence: u32,
}
```

Claimed-lease fields used to populate a [`JobInvocation`](#jobinvocation).

**Fields**

- `job_id`: Durable job id (`jobs.id`).
- `attempt`: 1-based attempt counter from the claim.
- `generation`: Lease generation from the successful claim.
- `dedup_key`: Queue dedup key; stable across reclaim of the same command.
- `deadline_unix_ms`: Lease expiry as UTC unix milliseconds (guest deadline hint).
- `checkpoint`: Checkpoint restored from a prior [`JobOutcome::Suspended`](#joboutcome).
- `invocation_sequence`: Resume ordinal from the durable payload; distinct from [`Self::attempt`](#jobinvocationlease).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### ListOptions

```rust
pub struct ListOptions {
    pub prefix: String,
    pub cursor: Option<String>,
    pub limit: u32,
}
```

Paginated list request.

**Fields**

- `prefix`: Key prefix filter.
- `cursor`: Opaque continuation token.
- `limit`: Requested page size (0 = default).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ListPage

```rust
pub struct ListPage {
    pub objects: Vec<ObjectInfo>,
    pub next_cursor: Option<String>,
}
```

One page of keys.

Cursors are opaque. Missing/stale cursors must return `invalid_cursor`,
never silently restart at page one. Concurrent mutation is weakly
consistent unless a backend snapshot is documented.

**Fields**

- `objects`: Objects in this page.
- `next_cursor`: Continuation token; `None` when this is the last page.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ObjectInfo

```rust
pub struct ObjectInfo {
    pub key: String,
    pub size: u64,
}
```

Object listing entry.

**Fields**

- `key`: Object key.
- `size`: Size in bytes.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ObjectMetadata

```rust
pub struct ObjectMetadata {
    pub key: String,
    pub size: u64,
    pub content_type: Option<String>,
    pub etag: Option<String>,
    pub sha256: Option<Vec<u8>>,
}
```

Metadata without a body.

**Fields**

- `key`: Object key.
- `size`: Size in bytes.
- `content_type`: MIME type when known.
- `etag`: Opaque etag when the backend provides one.
- `sha256`: SHA-256 digest when computed.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OidcClientTemplate

```rust
pub struct OidcClientTemplate {
    pub client_id: String,
    pub display_name: String,
    pub callback_path: String,
    pub public_client: bool,
    pub default_scopes: Vec<String>,
    pub issue_refresh_token: bool,
    pub origin_config_key: String,
}
```

Plugin-declared Bookclerk-as-IdP relying-party template.

The host materializes `oidc_clients` rows. Plugins never mint tokens.
`origin_config_key` is a dotted config path such as
`integrations.audiobookshelf.base_url`.

**Fields**

- `client_id`: Stable OAuth `client_id` registered with Bookclerk's authorization server.
- `display_name`: Operator-facing card title.
- `callback_path`: Path appended to the plugin origin (e.g. `/auth/openid/callback`).
- `public_client`: When true, the client is public PKCE (no secret).
- `default_scopes`: Scopes offered on first materialization (`openid` / `profile` typical).
- `issue_refresh_token`: When true, `/oidc/token` may issue refresh tokens for new rows.
- `origin_config_key`: Dotted config key that supplies the player origin.

#### OidcClientTemplate::scopes_or_default

```rust
pub fn scopes_or_default(&self) -> Vec<String>
```

Scopes used when the guest omitted `defaultScopes`.

#### OidcClientTemplate::display_name_or_id

```rust
pub fn display_name_or_id(&self) -> &str
```

Card title, falling back to `client_id`.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginDescribe

```rust
pub struct PluginDescribe {
    pub api_version: u32,
    pub id: String,
    pub display_name: Option<String>,
    pub rpc_features: Vec<String>,
    pub scalar_limits: ScalarLimitsDto,
    pub capabilities: PluginCapabilities,
    pub portal_auth_mode: PortalAuthMode,
    pub password_env_var: Option<String>,
    pub aliases: Vec<String>,
    pub sort_key: u32,
    pub brand: Option<Brand>,
    pub config_options: Vec<ConfigOption>,
    pub cli: CliSchema,
}
```

Guest identity returned by `PluginWorker.describe` (the adapter merges the manifest projection over the author `describe()`).

Every field is a typed Cap'n Proto field of `PluginDescribe`; there is no
side-channel JSON. Storefront UI extras (`brand`, `config_options`,
`portal_auth_mode`, ...) are empty / default for kinds that do not use them.

**Fields**

- `api_version`: Must equal [`crate::limits::PRODUCT_API_VERSION`](#product_api_version).
- `id`: Plugin id matching `plugin.toml`.
- `display_name`: Optional UI name.
- `rpc_features`: Accepted RPC features (intersection with the host offer).
- `scalar_limits`: Effective numeric limits.
- `capabilities`: Typed capability declaration (entrypoints, event consumers/producers,
  jobs, database and named bindings). The host compares it with the
  verified `plugin.toml` and the operator grant; widening is rejected.
- `portal_auth_mode`: Portal Accounts connect mode for storefronts.
- `password_env_var`: Reserved wire field: hosts ignore it. Passwords reach a guest only as
  sealed secrets from Accounts, never from the environment.
- `aliases`: Alternate ids accepted for config / CLI targeting.
- `sort_key`: UI sort weight among peers of the same kind; lower sorts first.
- `brand`: Portal brand; `None` when the guest has no brand (wire `brand.id` is
  empty) and the host renders a neutral fallback.
- `config_options`: Discoverable config option groups for source UIs.
- `cli`: Embedded CLI schema (same shape as `cliDescribe`); empty when unused.

#### PluginDescribe::has_entrypoint

```rust
pub fn has_entrypoint(&self, entrypoint: Entrypoint) -> bool
```

True when the guest exports the named [`Entrypoint`](generated.md#entrypoint).

#### PluginDescribe::consumes_event

```rust
pub fn consumes_event(&self, event_type: &str) -> bool
```

True when the guest's default entrypoint consumes `event_type`.

#### PluginDescribe::runs_job

```rust
pub fn runs_job(&self, job_type: &str) -> bool
```

True when the guest's default entrypoint runs jobs of `job_type`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PutResult

```rust
pub struct PutResult {
    pub key: String,
    pub bytes_written: u64,
    pub etag: Option<String>,
    pub sha256: Option<Vec<u8>>,
}
```

Result of a streamed put.

**Fields**

- `key`: Object key written.
- `bytes_written`: Bytes accepted.
- `etag`: Backend etag when available.
- `sha256`: Digest of the written body when computed.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueryPage

```rust
pub struct QueryPage {
    pub rows_json: String,
    pub next_cursor: Option<String>,
}
```

Bounded query page for host-internal SeaORM session helpers (not Cap'n ABI).

**Fields**

- `rows_json`: JSON-encoded rows.
- `next_cursor`: Continuation token.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScalarLimitsDto

```rust
pub struct ScalarLimitsDto {
    pub max_scalar_bytes: u32,
    pub max_stream_window_bytes: u32,
    pub max_list_page: u32,
}
```

Wire form of [`ScalarLimits`](#scalarlimits).

**Fields**

- `max_scalar_bytes`: Max ordinary RPC value size in bytes.
- `max_stream_window_bytes`: Max `ByteSource.pull` window in bytes.
- `max_list_page`: Max objects per list page.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `From<ScalarLimits>`, `From<ScalarLimitsDto>`, `PartialEq`, `Serialize`

### WriteOptions

```rust
pub struct WriteOptions {
    pub content_type: Option<String>,
    pub content_length: Option<u64>,
    pub sha256: Option<Vec<u8>>,
    pub commit_token: Option<String>,
    pub stage_only: bool,
}
```

Write options for `Destination.put`.

**Fields**

- `content_type`: MIME type.
- `content_length`: Known length; omit when unknown.
- `sha256`: Expected SHA-256 of the body.
- `commit_token`: Destination-side stage-and-publish token. Empty means a one-shot put.
- `stage_only`: When true, `put` stages remotely and does not publish until `commit`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### PluginMigrationOp

```rust
pub enum PluginMigrationOp {
    Schema(String),
    Data(String),
}
```

One already-separated BookclerkSQL operation in a registered migration.

**Variants**

- `Schema`: Admitted schema DDL (`CREATE` / `DROP` / index).
- `Data`: Admitted data DML (`INSERT` / `UPDATE` / `DELETE`).

#### PluginMigrationOp::sql

```rust
pub fn sql(&self) -> &str
```

Canonical BookclerkSQL for this operation.

#### PluginMigrationOp::is_schema

```rust
pub fn is_schema(&self) -> bool
```

True when this operation is classified as schema DDL.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OverflowDialect

```rust
pub enum OverflowDialect {
    SqliteFamily,
    Postgres,
}
```

Engine family for overflow derived-table syntax and i64::MIN CAST.

**Variants**

- `SqliteFamily`: SQLite / D1: correlated `FROM (SELECT …) _bc_ov`, `INTEGER` min CAST.
- `Postgres`: PostgreSQL: `LATERAL` + `BIGINT` min CAST.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`

### IntegerArithKind

```rust
pub enum IntegerArithKind {
    Add,
    Sub,
    Mul,
    Abs,
}
```

INTEGER `+` `-` `*` `abs` site lowered to overflow → NULL.

**Variants**

- `Add`: `lhs + rhs`
- `Sub`: `lhs - rhs`
- `Mul`: `lhs * rhs`
- `Abs`: `abs(arg)` (`lhs` is the argument span)

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SchemaAction

```rust
pub enum SchemaAction {
    None,
    Create { schema: Box<CreateTableSchema>, fingerprint: String, noop: bool },
    Drop { table: String },
}
```

CREATE/DROP action recorded on a proof.

**Variants**

- `None`: Not DDL.
- `Create`: `CREATE TABLE` — `noop` when the durable fingerprint already matches.
  - `schema`: Parsed SQL-v1 table schema (columns, nullability, identity, constraints).
  - `fingerprint`: Structured schema fingerprint (hex SHA-256).
  - `noop`: True when catalog already has this exact fingerprint.
- `Drop`: `DROP TABLE`.
  - `table`: Folded table name.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LikePatternSrc

```rust
pub enum LikePatternSrc {
    Literal(String),
    Bind(usize),
    Unproven,
    Null,
}
```

Proven LIKE pattern sources in one canonical statement.

**Variants**

- `Literal`: `'literal'` (possibly concatenated with `||`).
- `Bind`: Positional `?` bind (0-based bind index in this statement).
- `Unproven`: Pattern expression whose length cannot be proven.
- `Null`: SQL `NULL` (no pattern bytes).

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### EventResult

```rust
pub enum EventResult {
    Ack,
    Retry { retry_at_unix_ms: u64, reason: String },
    Reject { reason: String },
    DeadLetter { reason: String },
    Suspended { checkpoint_json: String, checkpoint_schema_version: u32, wake_at_unix_ms: u64, wake_on_event_type: String, wake_on_filter_json: String },
}
```

Per-event result of [`crate::EventConsumer::event`](#eventconsumerevent).

**Variants**

- `Ack`: Event processed.
- `Retry`: Transient failure; retry after `retry_at_unix_ms`.
  - `retry_at_unix_ms`: UTC unix-ms hint.
  - `reason`: Operator-facing reason.
- `Reject`: Permanent rejection.
  - `reason`: Operator-facing reason.
- `DeadLetter`: Poison; do not redeliver.
  - `reason`: Operator-facing reason.
- `Suspended`: Durable sleep: persist checkpoint, release the process, resume later.
  - `checkpoint_json`: Bounded checkpoint JSON (must be ≤ [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes)).
  - `checkpoint_schema_version`: Checkpoint schema version.
  - `wake_at_unix_ms`: UTC unix-ms wake hint.
  - `wake_on_event_type`: Event type that can wake this sleep; empty = timestamp-only.
  - `wake_on_filter_json`: Host-owned payload object filter JSON; empty = type only.

#### EventResult::from_json_value

```rust
pub fn from_json_value(value: &Value) -> Result<Self>
```

Decode JSON `{"kind":"ack"|…}`. Missing or unknown `kind` is an error.

##### Errors

Returns [`crate::PluginErrorCode::InvalidParams`](error.md#pluginerrorcode) when the payload is not
a tagged [`EventResult`](#eventresult), or [`crate::PluginErrorCode::PayloadTooLarge`](error.md#pluginerrorcode)
when a `suspended` checkpoint exceeds [`MAX_CHECKPOINT_BYTES`](#max_checkpoint_bytes).

#### EventResult::from_json_str

```rust
pub fn from_json_str(raw: &str) -> Result<Self>
```

Decode a JSON object string as [`EventResult`](#eventresult).

##### Errors

Same as [`Self::from_json_value`](#eventresultfrom_json_value).

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### JobOutcome

```rust
pub enum JobOutcome {
    Completed { message: String, bytes_copied: u64 },
    Retryable { message: String, retry_after_unix_ms: Option<u64> },
    Rejected { message: String },
    Cancelled { message: String },
    Suspended { checkpoint: JobCheckpoint, wake_at_unix_ms: u64 },
}
```

Outcome of [`crate::JobRunner::job`](#jobrunnerjob).

Suspension is durable only after Bookclerk atomically commits the fenced
outcome. Open streams and process memory do not survive.

**Variants**

- `Completed`: Handler finished successfully.
  - `message`: Operator-facing summary.
  - `bytes_copied`: Bytes copied when this was a stream copy.
- `Retryable`: Transient failure; host may retry after `retry_after_unix_ms`.
  - `message`: Operator-facing summary.
  - `retry_after_unix_ms`: Optional UTC unix-ms hint for the next attempt.
- `Rejected`: Permanent rejection (do not retry the same payload).
  - `message`: Operator-facing summary.
- `Cancelled`: Cancelled by host fence / guest abort.
  - `message`: Operator-facing summary.
- `Suspended`: Durable pause; resume with `checkpoint` after `wake_at_unix_ms`.
  - `checkpoint`: Checkpoint to restore on wake.
  - `wake_at_unix_ms`: UTC unix-ms wake hint.

#### JobOutcome::is_terminal

```rust
pub fn is_terminal(&self) -> bool
```

True when the host should treat the invocation as finished.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Functions

### capnp_u32_len

```rust
pub fn capnp_u32_len(len: usize, what: &str) -> Result<u32>
```

Converts a Rust list length into a Cap'n Proto `UInt32` list size.

#### Errors

Returns [`PluginError::payload_too_large`](error.md#pluginerrorpayload_too_large) when `len` cannot fit in `u32`.

### plugin_migration_registration_bytes

```rust
pub fn plugin_migration_registration_bytes(migrations: &[PluginMigration]) -> usize
```

Aggregate UTF-8 bytes of ids plus SQL across a complete registration.

### require_plugin_migration_json_lists

```rust
pub fn require_plugin_migration_json_lists(value: &Value) -> Result<()>
```

Rejects JSON `databaseMigrations` list sizes before deserializing operations.

Walks nested `operations` array lengths only; does not construct
[`PluginMigration`](#pluginmigration) values. Call this before `serde_json::from_value`.

#### Errors

Returns [`PluginError::payload_too_large`](error.md#pluginerrorpayload_too_large) when migration count, operations
per migration, or total operations exceed ABI limits.

### require_plugin_migration_registration

```rust
pub fn require_plugin_migration_registration(migrations: &[PluginMigration]) -> Result<()>
```

Rejects a plugin migration registration that exceeds RPC resource limits.

Call this before encoding a Cap'n Proto / JSON reply and while decoding,
before semantic BookclerkSQL proof. Limits:

- migration count: [`MAX_LIST_PAGE`](#max_list_page)
- operations per migration: [`MAX_PLUGIN_MIGRATION_OPS`](#max_plugin_migration_ops)
- total operations: [`MAX_PLUGIN_MIGRATION_TOTAL_OPS`](#max_plugin_migration_total_ops)
- each SQL text: [`MAX_SCALAR_BYTES`](#max_scalar_bytes)
- aggregate id + SQL bytes: [`MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES`](#max_plugin_migration_registration_bytes)

Total operations are checked before walking SQL text so a registration of
many tiny operations is rejected without treating the 256 KiB text budget
as a heap bound.

#### Errors

Returns [`PluginError::payload_too_large`](error.md#pluginerrorpayload_too_large) when any limit is exceeded.

### apply_integer_overflow

```rust
pub fn apply_integer_overflow(
    dialect: OverflowDialect,
    sql: &str,
    sites: &[IntegerArithSite],
) -> Result<String>
```

Wrap each INTEGER `+`/`-`/`*`/`abs` site in a portable overflow CASE.

Sites are applied **innermost-first** (`full.end`, then reverse `full.start`)
so right-nested `1 + abs(n)` rewrites the `abs` call before the outer add.
After each wrap, remaining (outer) sites shift by the byte count the wrap
added or removed; a wrap drops comments between operands, so it can be
shorter than its site.

#### Errors

Returns when a site is empty, past `sql`, not on a UTF-8 boundary, cannot
be wrapped as the recorded operator, or has a boundary strictly inside an
inner site it does not enclose.

### assert_proof_matches_sql

```rust
pub fn assert_proof_matches_sql(proof: &ResolvedStatement, sql: &str) -> Result<()>
```

Require `proof` to be bound to `sql`.

#### Errors

Returns when the hashes differ (transformed or reindexed SQL).

### admitted_bookclerk_sql_samples

```rust
pub fn admitted_bookclerk_sql_samples(seed: u64, count: usize) -> Vec<String>
```

Grammar-aware BookclerkSQL samples for fuzz corpora and differential tests.

Statements stay inside the SQL-v1 grammar (canonical `?`, `LIKE`,
`INSERT OR IGNORE`). TEXT literals are portable UTF-8 without U+0000.
LIKE patterns stay within [`D1_PORTABLE_LIKE_PATTERN_BYTES`](#d1_portable_like_pattern_bytes).

### canonical_statements_checksum

```rust
pub fn canonical_statements_checksum(statements: &[&str]) -> String
```

Length-prefixed SHA-256 of an ordered canonical statement list.

Statement boundaries are part of the digest: joining with `;` then hashing
would collide with literals such as `DEFAULT 'a;b'`.

### glob_expanded_like_pattern_bytes

```rust
pub fn glob_expanded_like_pattern_bytes(like_pattern: &str) -> usize
```

UTF-8 byte length of a sqlite-family GLOB pattern after Bookclerk LIKE→GLOB
metacharacter escaping (`[` `*` `?` become three bytes).

D1's physical GLOB/LIKE pattern cap is 50 bytes. Adapters that perform this
expansion must advertise [`super::DbCapabilities::max_pattern_bytes`](db_execute.md#dbcapabilities) small
enough that the expanded pattern still fits.

### like_pattern_sources

```rust
pub fn like_pattern_sources(sql: &str) -> Result<Vec<LikePatternSrc>>
```

Collects LIKE / NOT LIKE pattern sources with proven lengths when possible.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a string, quoted identifier,
or block comment is unterminated.

### require_function_args_within

```rust
pub fn require_function_args_within(sql: &str, max_function_args: u32) -> Result<()>
```

Enforces [`crate::DbCapabilities::max_function_args`](db_execute.md#dbcapabilities) on non-chunkable helpers.

`min` / `max` / `coalesce` are omitted: adapters nest them so each physical
call stays inside the advertised cap. `json_object` is enforced here: the
portable maximum is already [`crate::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#sql_v1_json_object_max_args).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a non-chunkable call exceeds
the cap or the cap is unspecified.

### require_like_patterns_within

```rust
pub fn require_like_patterns_within(
    sql: &str,
    parameters: &[DbValue],
    max_pattern_bytes: u32,
) -> Result<()>
```

Enforces [`crate::DbCapabilities::max_pattern_bytes`](db_execute.md#dbcapabilities) on LIKE patterns.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a proven pattern exceeds the
cap or a LIKE pattern length cannot be proven.

### require_portable_text

```rust
pub fn require_portable_text(text: &str) -> Result<()>
```

Rejects BookclerkSQL `TEXT` that contains U+0000.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when `text` contains NUL.

### require_portable_text_binds

```rust
pub fn require_portable_text_binds(parameters: &[DbValue]) -> Result<()>
```

Rejects U+0000 in every BookclerkSQL `TEXT` bind.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a [`crate::DbValue::Text`](db_value.md#dbvalue) bind
contains NUL.

### sql_v1_function_calls

```rust
pub fn sql_v1_function_calls(sql: &str) -> Result<Vec<SqlFnCall>>
```

Collects helper calls in code spans (strings/comments are skipped).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a string, quoted identifier,
or block comment is unterminated.

### sql_v1_helper_is_chunkable

```rust
pub fn sql_v1_helper_is_chunkable(name: &str) -> bool
```

Helpers adapters may nest so each physical call stays inside
[`crate::DbCapabilities::max_function_args`](db_execute.md#dbcapabilities).

`json_object` is not chunkable: portable SQL-v1 already caps it at
[`crate::SQL_V1_JSON_OBJECT_MAX_ARGS`](sql_types.md#sql_v1_json_object_max_args) (D1's physical limit). Nesting via
`json_patch` is not equivalent (JSON Merge Patch deletes nulls).

### sql_v1_pack_statements

```rust
pub fn sql_v1_pack_statements(sql: &str) -> Result<Vec<String>>
```

Splits a BookclerkSQL pack into statements using the SQL-v1 lexer.

Semicolons inside string literals, quoted identifiers, line comments, and
block comments do not end a statement. Empty fragments are dropped.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when quotes or comments are
unterminated, or when a fragment is not a single SQL-v1 statement.

### sqlite_family_like_divmod_insert_upper_bound

```rust
pub const fn sqlite_family_like_divmod_insert_upper_bound(canonical_len: usize) -> usize
```

Upper bound on sqlite-family mechanical lowering length for a canonical
statement of `canonical_len` bytes (`div`/`mod` NULLIF, `INSERT OR IGNORE`,
LIKE→GLOB). Does not include proof-directed INTEGER overflow wraps.

LIKE→GLOB is the densest per-byte expansion among mechanical rewrites
(`LIKE` is 4 bytes; overhead is [`LIKE_GLOB_REWRITE_OVERHEAD`](#like_glob_rewrite_overhead)). The count
`canonical_len / 4` is the maximum number of `LIKE` keywords that can appear
in `canonical_len` bytes — not an expansion-factor guess. Remaining bytes
after packing as many `LIKE` keywords as possible are charged as `/` `%`
NULLIF wraps (1-byte operators).

Proof-directed INTEGER overflow CASE wraps are applied from typed proofs
before this mechanical bound; they are not a function of payload length
alone.

### sqlite_family_mechanical_len_upper_bound

```rust
pub fn sqlite_family_mechanical_len_upper_bound(sql: &str) -> Result<usize>
```

Mechanical sqlite-family length bound for one canonical statement, counting
actual `LIKE` occurrences, `/` `%` operators in code spans, and at most one
`INSERT OR IGNORE` rewrite.

Always `<=` [`sqlite_family_like_divmod_insert_upper_bound`](#sqlite_family_like_divmod_insert_upper_bound)`(sql.len())`
for well-formed packs: the length formula packs the densest rewrite into
every 4-byte window, while this preflight charges only constructs that
lowering actually rewrites. INTEGER overflow wraps are applied from typed
proofs *before* this mechanical bound.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when quotes or block comments are
unterminated.

### text_contains_nul

```rust
pub fn text_contains_nul(text: &str) -> bool
```

True when `text` contains U+0000.

### canonical_execute_request_hash

```rust
pub fn canonical_execute_request_hash(req: &ExecuteRequest) -> Result<String>
```

SHA-256 hex of the Cap'n-encoded request with transport metadata cleared.

`operationId`, `requestHash`, and `deadlineUnixMs` are not part of the
digest: a retry of the same mutation may mint a remaining deadline without
hash-conflicting. The trusted host stamps `requestHash` after validation
so guests cannot collide unrelated mutations on one idempotency key.

#### Errors

Returns [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### decode_db_value_bytes

```rust
pub fn decode_db_value_bytes(bytes: &[u8]) -> Result<DbValue>
```

Decodes a standalone Cap'n `DbValue` message.

#### Errors

Returns when the buffer is not a valid unpacked `DbValue`.

### decode_execute_request_bytes

```rust
pub fn decode_execute_request_bytes(bytes: &[u8]) -> Result<ExecuteRequest>
```

Decodes a standalone Cap'n `ExecuteRequest` message.

#### Errors

Returns when the buffer is not a valid unpacked `ExecuteRequest`.

### decode_execute_result_reply_bytes

```rust
pub fn decode_execute_result_reply_bytes(bytes: &[u8]) -> Result<ExecuteReply>
```

Decodes a standalone Cap'n `ExecuteAtomicReply` message.

#### Errors

Returns the guest [`PluginError`](error.md#pluginerror) on `err`, or a decode failure on `ok`.

### encoded_db_value_bytes

```rust
pub fn encoded_db_value_bytes(v: &DbValue) -> Result<Vec<u8>>
```

Encodes a standalone Cap'n `DbValue` message (unpacked stream).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a [`DbValue::Text`](db_value.md#dbvalue) contains
U+0000, or [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### encoded_execute_reply_bytes

```rust
pub fn encoded_execute_reply_bytes(reply: &ExecuteReply) -> Result<Vec<u8>>
```

Encodes a standalone Cap'n `ExecuteReply` message (unpacked stream).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a TEXT cell contains U+0000,
or [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### encoded_execute_request_bytes

```rust
pub fn encoded_execute_request_bytes(req: &ExecuteRequest) -> Result<Vec<u8>>
```

Encodes a standalone Cap'n `ExecuteRequest` message (unpacked stream).

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when SQL or TEXT binds contain
U+0000, or [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### encoded_execute_result_reply_bytes

```rust
pub fn encoded_execute_result_reply_bytes(
    outcome: Result<ExecuteReply>,
) -> Result<Vec<u8>>
```

Encodes a standalone Cap'n `ExecuteAtomicReply` (`ok` or `err`).

#### Errors

Returns [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### encoded_statement_result_bytes

```rust
pub fn encoded_statement_result_bytes(stmt: &StatementResult) -> Result<Vec<u8>>
```

Encodes a standalone Cap'n `StatementResult` message (unpacked stream).

Used to enforce negotiated `maxResultBytes` against the actual wire size of
one statement before COMMIT.

#### Errors

Returns [`PluginError::invalid_params`](error.md#pluginerrorinvalid_params) when a TEXT cell contains U+0000,
or [`PluginError::internal`](error.md#pluginerrorinternal) when the message cannot be serialized.

### negotiate_rpc_features

```rust
pub fn negotiate_rpc_features(
    host_offers: &[&str],
    guest_accepts: &[String],
) -> Result<Vec<String>>
```

Intersects host-offered and guest-accepted feature lists.

#### Errors

Returns a message when a required feature is missing or the guest offers an
unsafe combination (streams without scalar limits).

### read_all

```rust
pub async fn read_all(body: Pin<Box<dyn AsyncRead + Send>>) -> Result<Vec<u8>>
```

Reads a small transferred body, stopping at [`crate::MAX_SCALAR_BYTES`](#max_scalar_bytes).

This is not a media reader. A longer body is [`PluginError::payload_too_large`](error.md#pluginerrorpayload_too_large)
and the extra bytes are not retained.

#### Errors

Returns [`PluginError::payload_too_large`](error.md#pluginerrorpayload_too_large) or an I/O error mapped as
[`PluginError::internal`](error.md#pluginerrorinternal).

### stream_copy_keys

```rust
pub async fn stream_copy_keys(
    input: &dyn Source,
    output: &dyn Destination,
    spec: &StreamCopySpec,
    invocation: &JobInvocation,
    progress: &dyn ProgressSink,
    cancel: &dyn Cancellation,
) -> Result<JobOutcome>
```

Copies `from` → `to` through granted source/destination streams.

Bytes are staged under a retry-stable `commit_token` then committed after a
best-effort fence check. Publication is at-least-once and idempotent on that
token. Cancellation, a failed cancellation poll, or put/commit failure
aborts the stage. A poll error is not treated as "still running".

#### Errors

Returns a plugin error when open, put, commit, progress reporting, or
cancellation polling fails.

### byte_source_from_async_read

```rust
pub fn byte_source_from_async_read(
    reader: Pin<Box<dyn AsyncRead + Send>>,
    window: u32,
) -> Client
```

Returns a `ByteSource` client that pulls from `reader`.

### connect_plugin

```rust
pub fn connect_plugin<R, W>(
    reader: R,
    writer: W,
    window: u32,
) -> (PluginClient, RpcSystem<Side>)
where
    R: AsyncRead + Unpin + 'static,
    W: AsyncWrite + Unpin + 'static,
```

Connects as the client side of a two-party vat.

Must run inside a `tokio::task::LocalSet`. The returned [`RpcSystem`](https://docs.rs/capnp_rpc/latest/capnp_rpc/struct.RpcSystem.html) must be
spawned with `tokio::task::spawn_local`.

### pull_byte_source_to_writer

```rust
pub async fn pull_byte_source_to_writer<W: AsyncWrite + Unpin>(
    source: Client,
    writer: &mut W,
    window: u32,
) -> Result<u64>
```

Pulls `source` into `writer` using bounded windows.

A failed `ByteSource.pull` is returned as [`PluginError`](error.md#pluginerror) — never as a clean
EOF. Destinations must abort rather than commit.

#### Errors

Returns a plugin error when the stream or writer fails.

### serve_plugin

```rust
pub async fn serve_plugin<R, W>(
    plugin: Arc<dyn PluginWorker>,
    reader: R,
    writer: W,
    window: u32,
) -> Result<()>
where
    R: AsyncRead + Unpin + 'static,
    W: AsyncWrite + Unpin + 'static,
```

Serves `plugin` as the bootstrap object on a two-party vat over `reader`/`writer`.

Must run inside a `tokio::task::LocalSet`.

#### Errors

Returns a plugin error when the vat fails.

### serve_plugin_stdio

```rust
pub async fn serve_plugin_stdio(plugin: Arc<dyn PluginWorker>, window: u32) -> Result<()>
```

Serves `plugin` on stdin/stdout. Must run on a current-thread runtime + `LocalSet`.

#### Errors

Returns a plugin error when the vat fails.

## Constants

### MAX_PLUGIN_MIGRATION_ID_BYTES

```rust
pub const MAX_PLUGIN_MIGRATION_ID_BYTES: usize = 128;
```

Maximum UTF-8 bytes of one plugin-chosen migration id.

### PLUGIN_MIGRATIONS_TABLE

```rust
pub const PLUGIN_MIGRATIONS_TABLE: &str = "bookclerk_plugin_migrations";
```

Host-private plugin-binding migration journal (not a plugin-owned table).

### PHYSICAL_STAR_COLUMN

```rust
pub const PHYSICAL_STAR_COLUMN: &str = "*";
```

Column name recorded for `SELECT *` / `alias.*` / `RETURNING *` on a physical table.

### D1_PHYSICAL_LIKE_GLOB_PATTERN_BYTES

```rust
pub const D1_PHYSICAL_LIKE_GLOB_PATTERN_BYTES: u32 = 50;
```

Physical D1 LIKE/GLOB pattern cap in bytes.

### D1_PORTABLE_LIKE_PATTERN_BYTES

```rust
pub const D1_PORTABLE_LIKE_PATTERN_BYTES: u32 = 16;
```

Conservative portable LIKE pattern cap for D1 while LIKE lowers to GLOB.

Cloudflare D1 allows 50 bytes in a LIKE/GLOB pattern. Worst-case expansion
of `[` / `*` / `?` is 3×, so `floor(50/3) = 16`.

### LIKE_GLOB_PATTERN_WRAP_BYTES

```rust
pub const LIKE_GLOB_PATTERN_WRAP_BYTES: usize = _;
```

Bytes added around a LIKE pattern by [`LIKE_GLOB_WRAP_PREFIX`](#like_glob_wrap_prefix) /
[`LIKE_GLOB_WRAP_SUFFIX`](#like_glob_wrap_suffix) (the pattern itself is unchanged).

### LIKE_GLOB_REWRITE_OVERHEAD

```rust
pub const LIKE_GLOB_REWRITE_OVERHEAD: usize = _;
```

Worst-case extra bytes per `LIKE` / `NOT LIKE` after sqlite-family GLOB wrapping.

### LIKE_GLOB_WRAP_PREFIX

```rust
pub const LIKE_GLOB_WRAP_PREFIX: &str = "replace(replace(replace(replace(replace((";
```

Prefix of the sqlite-family LIKE→GLOB `replace(...)` wrap (before the pattern).

Must stay identical to the GLOB wrap in `bookclerk-db-exec`.

### LIKE_GLOB_WRAP_SUFFIX

```rust
pub const LIKE_GLOB_WRAP_SUFFIX: &str = "), '[', '[[]'), '*', '[*]'), '?', '[?]'), '%', '*'), '_', '?')";
```

Suffix of the sqlite-family LIKE→GLOB `replace(...)` wrap (after the pattern).

### LIKE_TO_GLOB_KEYWORD_EXTRA

```rust
pub const LIKE_TO_GLOB_KEYWORD_EXTRA: usize = 1;
```

Extra byte when `LIKE` is immediately followed by its pattern (no trivia):
the rewrite emits `GLOB ` (5) in place of `LIKE` (4).

### SQLITE_FAMILY_DIV_MOD_NULLIF_EXTRA

```rust
pub const SQLITE_FAMILY_DIV_MOD_NULLIF_EXTRA: usize = 11;
```

Extra bytes for one `/` or `%` rewritten to `NULLIF(operand, 0)`.

### SQLITE_FAMILY_INSERT_OR_IGNORE_MAX_EXTRA

```rust
pub const SQLITE_FAMILY_INSERT_OR_IGNORE_MAX_EXTRA: usize = 256;
```

Conservative extra bytes for one `INSERT OR IGNORE` unique-conflict rewrite
(conflict clause plus optional `SELECT` source wrap).

### FEATURE_SCALAR_LIMITS

```rust
pub const FEATURE_SCALAR_LIMITS: &str = crate::plugin_capnp::FEATURE_SCALAR_LIMITS;
```

Guest honors scalar / stream-window / list-page caps.

### FEATURE_STORAGE_COPY

```rust
pub const FEATURE_STORAGE_COPY: &str = crate::plugin_capnp::FEATURE_STORAGE_COPY;
```

Guest implements server-side [`super::Destination::copy`](#destinationcopy).

### FEATURE_STREAMS

```rust
pub const FEATURE_STREAMS: &str = crate::plugin_capnp::FEATURE_STREAMS;
```

Media moves through transferred [`crate::roles::ByteRange`](#byterange) / `ByteSource` streams.

### MAX_EVENT_PAYLOAD_BYTES

```rust
pub const MAX_EVENT_PAYLOAD_BYTES: u32 = crate::plugin_capnp::MAX_EVENT_PAYLOAD_BYTES;
```

Maximum decoded size of a domain-event scalar payload (not a stream).

### MAX_KV_KEY_BYTES

```rust
pub const MAX_KV_KEY_BYTES: u32 = crate::plugin_capnp::MAX_KV_KEY_BYTES;
```

Longest `KvNamespace` key in UTF-8 bytes.

### MAX_KV_METADATA_BYTES

```rust
pub const MAX_KV_METADATA_BYTES: u32 = crate::plugin_capnp::MAX_KV_METADATA_BYTES;
```

Largest `KvNamespace` metadata JSON text in bytes.

### MAX_KV_VALUE_BYTES

```rust
pub const MAX_KV_VALUE_BYTES: u32 = crate::plugin_capnp::MAX_KV_VALUE_BYTES;
```

Largest `KvNamespace` value in bytes (a `[[kv_namespaces]]` row may lower
its own cap but never raise it past this).

### MAX_LIST_PAGE

```rust
pub const MAX_LIST_PAGE: u32 = crate::plugin_capnp::MAX_LIST_PAGE;
```

Maximum objects in one `Destination.list` page.

### MAX_PLUGIN_MIGRATION_OPS

```rust
pub const MAX_PLUGIN_MIGRATION_OPS: u32 = crate::plugin_capnp::MAX_PLUGIN_MIGRATION_OPS;
```

Maximum already-separated operations in one plugin-owned migration.

### MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES

```rust
pub const MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES: u32 = crate::plugin_capnp::MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES;
```

Maximum aggregate UTF-8 bytes of plugin migration ids plus SQL in one
`databaseMigrations` registration (Cap'n Proto framing is not counted).

### MAX_PLUGIN_MIGRATION_TOTAL_OPS

```rust
pub const MAX_PLUGIN_MIGRATION_TOTAL_OPS: u32 = crate::plugin_capnp::MAX_PLUGIN_MIGRATION_TOTAL_OPS;
```

Maximum already-separated operations across one `databaseMigrations`
registration (independent of the per-migration cap and of id+SQL UTF-8 bytes).

### MAX_SCALAR_BYTES

```rust
pub const MAX_SCALAR_BYTES: u32 = crate::plugin_capnp::MAX_SCALAR_BYTES;
```

Maximum decoded size of an ordinary RPC scalar value (not a stream window).

### MAX_STREAM_WINDOW_BYTES

```rust
pub const MAX_STREAM_WINDOW_BYTES: u32 = crate::plugin_capnp::MAX_STREAM_WINDOW_BYTES;
```

Maximum bytes returned by one `ByteSource.pull` (flow-control window).

### PRODUCT_API_VERSION

```rust
pub const PRODUCT_API_VERSION: u32 = crate::plugin_capnp::API_VERSION;
```

Product ABI version (`apiVersion` / `plugin.toml` `api_version`).

### JSON_MEDIA_TYPE

```rust
pub const JSON_MEDIA_TYPE: &str = "application/json";
```

Media type of a JSON [`ExtensibleConfig`](#extensibleconfig) payload.

### MAX_CHECKPOINT_BYTES

```rust
pub const MAX_CHECKPOINT_BYTES: u32 = crate::plugin_capnp::MAX_CHECKPOINT_BYTES;
```

Maximum checkpoint payload size (bytes).

### PLUGIN_TOML_SCHEMA_JSON

```rust
pub const PLUGIN_TOML_SCHEMA_JSON: &str = "{\n  \"$schema\": \"https://json-schema.org/draft/2020-12/schema\",\n  \"$id\": \"https://bookclerk.dev/schemas/plugin-toml.json\",\n  \"title\": \"BookclerkPluginToml\",\n  \"description\": \"Install descriptor for a Bookclerk plugin (plugin.toml), api_version = 3. Keys use TOML/snake_case. There is no plugin `kind`; handler family is derived from entrypoints and triggers. The `id` is a display/CLI alias, not a globally unique security identity. Durable identity is the host-computed provenance-qualified PluginKey. capabilities.network.domains is workerd-only (hostname allowlist for isolate fetch()). Native guests omit domains and declare capabilities.network.tcp for mediated connect(); ambient AF_INET is denied.\",\n  \"type\": \"object\",\n  \"additionalProperties\": false,\n  \"required\": [\"api_version\", \"id\", \"capabilities\"],\n  \"properties\": {\n    \"api_version\": { \"type\": \"integer\", \"enum\": [3] },\n    \"id\": {\n      \"type\": \"string\",\n      \"minLength\": 2,\n      \"maxLength\": 32,\n      \"pattern\": \"^[a-z0-9]+(?:_[a-z0-9]+)*$\",\n      \"description\": \"Display / CLI alias. Not globally unique and not a privilege. Lowercase letters, digits, and single underscores only (2\u{2013}32 chars); no leading/trailing or doubled `_`. When two installs share an alias, address them with a provenance-qualified PluginKey.\"\n    },\n    \"name\": { \"type\": \"string\" },\n    \"version\": { \"type\": \"string\" },\n    \"logo\": {\n      \"type\": \"string\",\n      \"description\": \"Optional Settings/UI logo: https:// or http:// URL (browser loads directly), or a relative image path under the plugin install root (host serves via GET /api/plugins/{id}/logo when the alias is unambiguous). Embedded paths must not contain \'..\' and must end with .png/.jpg/.jpeg/.gif/.webp/.svg/.ico.\"\n    },\n    \"runtime\": {\n      \"type\": \"string\",\n      \"enum\": [\"native\", \"workerd\"],\n      \"default\": \"native\"\n    },\n    \"command\": { \"type\": \"string\", \"minLength\": 1 },\n    \"args\": {\n      \"type\": \"array\",\n      \"items\": { \"type\": \"string\" }\n    },\n    \"entrypoints\": {\n      \"type\": \"array\",\n      \"description\": \"Named capabilities this plugin exports from PluginWorker.open.\",\n      \"items\": {\n        \"type\": \"string\",\n        \"enum\": [\"storefront\", \"storage\", \"databaseAdapter\", \"remoteLibrary\", \"cli\", \"oidc\"]\n      }\n    },\n    \"workerd\": { \"$ref\": \"#/$defs/WorkerdRuntime\" },\n    \"modules\": {\n      \"type\": \"array\",\n      \"items\": { \"$ref\": \"#/$defs/ModuleSpec\" }\n    },\n    \"triggers\": { \"$ref\": \"#/$defs/Triggers\" },\n    \"events\": { \"$ref\": \"#/$defs/Events\" },\n    \"databases\": {\n      \"type\": \"array\",\n      \"description\": \"Named plugin-owned database bindings (Workers-style).\",\n      \"maxItems\": 8,\n      \"items\": { \"$ref\": \"#/$defs/DatabaseBinding\" }\n    },\n    \"vars\": {\n      \"type\": \"object\",\n      \"description\": \"Operator config binding (`CONFIG`). Declares that the guest receives CONFIG; values come from [[settings]] defaults and the plugin instance document. An empty table is the usual form.\"\n    },\n    \"settings\": {\n      \"type\": \"array\",\n      \"description\": \"Operator settings schema ([[settings]]): keys an instance document may set, their types, enum values, and defaults. The host validates documents against it and renders it in Settings. Non-secret entries require [vars]; type = \\\"secret\\\" entries require [secrets].\",\n      \"maxItems\": 32,\n      \"items\": { \"$ref\": \"#/$defs/SettingSpec\" }\n    },\n    \"secrets\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"kv_namespaces\": {\n      \"type\": \"array\",\n      \"description\": \"Durable KV bindings (default name KV) served from the library database: Workers KVNamespace-shaped get/put/delete/list with expiration and metadata, scoped to the plugin and the row\'s namespace. Distinct from [[databases]].\",\n      \"items\": { \"$ref\": \"#/$defs/KvNamespace\" }\n    },\n    \"queues\": { \"$ref\": \"#/$defs/Queues\" },\n    \"work_fs\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"oauth\": { \"$ref\": \"#/$defs/NamedBinding\" },\n    \"capabilities\": { \"$ref\": \"#/$defs/Capabilities\" },\n    \"cli\": { \"$ref\": \"#/$defs/CliSchema\" },\n    \"oidc\": { \"$ref\": \"#/$defs/OidcManifest\" }\n  },\n  \"allOf\": [\n    {\n      \"if\": {\n        \"properties\": { \"runtime\": { \"const\": \"workerd\" } },\n        \"required\": [\"runtime\"]\n      },\n      \"then\": { \"required\": [\"workerd\"] }\n    },\n    {\n      \"if\": {\n        \"anyOf\": [\n          { \"properties\": { \"runtime\": { \"const\": \"native\" } } },\n          { \"not\": { \"required\": [\"runtime\"] } }\n        ]\n      },\n      \"then\": {\n        \"required\": [\"command\"],\n        \"properties\": {\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"properties\": {\n                  \"domains\": false\n                }\n              }\n            }\n          }\n        }\n      }\n    },\n    {\n      \"if\": {\n        \"properties\": {\n          \"runtime\": { \"const\": \"workerd\" },\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"properties\": { \"mode\": { \"const\": \"outbound\" } },\n                \"required\": [\"mode\"]\n              }\n            },\n            \"required\": [\"network\"]\n          }\n        },\n        \"required\": [\"runtime\", \"capabilities\"]\n      },\n      \"then\": {\n        \"properties\": {\n          \"capabilities\": {\n            \"properties\": {\n              \"network\": {\n                \"required\": [\"domains\"],\n                \"properties\": {\n                  \"domains\": {\n                    \"type\": \"array\",\n                    \"minItems\": 1,\n                    \"items\": { \"type\": \"string\", \"minLength\": 1 }\n                  }\n                }\n              }\n            }\n          }\n        }\n      }\n    }\n  ],\n  \"$defs\": {\n    \"Capabilities\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"network\"],\n      \"properties\": {\n        \"network\": { \"$ref\": \"#/$defs/NetworkCapabilities\" }\n      }\n    },\n    \"NetworkCapabilities\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"mode\"],\n      \"properties\": {\n        \"mode\": { \"type\": \"string\", \"enum\": [\"deny\", \"outbound\"] },\n        \"domains\": {\n          \"type\": \"array\",\n          \"description\": \"Workerd-only: initial-request host allowlist for isolate egress. Forbidden on runtime=native. The operator may add destinations beyond this list; the guest cannot.\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        },\n        \"tcp\": {\n          \"type\": \"array\",\n          \"description\": \"Raw TCP connect grants (host + ports). Fetch domain grants do not imply TCP.\",\n          \"items\": { \"$ref\": \"#/$defs/TcpCapability\" }\n        },\n        \"allow_undeclared_public_redirects\": {\n          \"type\": \"boolean\",\n          \"description\": \"When true, fetch redirects may target undeclared public hosts. Address-space policy still applies.\"\n        },\n        \"address_cidrs\": {\n          \"type\": \"array\",\n          \"description\": \"Explicit CIDR grants beyond the public Internet. No blanket private=true.\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        }\n      }\n    },\n    \"TcpCapability\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"host\", \"ports\"],\n      \"properties\": {\n        \"host\": { \"type\": \"string\", \"minLength\": 1 },\n        \"ports\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 65535 },\n          \"minItems\": 1\n        }\n      }\n    },\n    \"Triggers\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"jobs\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"string\", \"minLength\": 1 }\n        }\n      }\n    },\n    \"Events\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"consumers\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/EventConsumer\" }\n        },\n        \"producers\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/EventProducer\" }\n        }\n      }\n    },\n    \"EventConsumer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"type\"],\n      \"properties\": {\n        \"type\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"pattern\": \"^[a-z][a-z0-9_]*$\"\n        },\n        \"schema_versions\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"integer\", \"minimum\": 1 },\n          \"minItems\": 1\n        },\n        \"supports_suspend\": { \"type\": \"boolean\", \"default\": false },\n        \"resource_class\": { \"type\": \"string\", \"enum\": [\"network\"], \"default\": \"network\" },\n        \"filter\": { \"type\": \"object\" },\n        \"max_retries\": { \"type\": \"integer\", \"minimum\": 0 }\n      }\n    },\n    \"EventProducer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"type\"],\n      \"properties\": {\n        \"type\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"pattern\": \"^[a-z][a-z0-9_]*$\"\n        },\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    },\n    \"Queues\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"description\": \"Workers Queues: producers are `env` send bindings, consumers are `queue(batch)` triggers. Queue names are `[a-z0-9][a-z0-9._-]*` (at most 63 chars); the `core.` and `host.` prefixes are reserved for the host.\",\n      \"properties\": {\n        \"producers\": { \"type\": \"array\", \"items\": { \"$ref\": \"#/$defs/QueueProducer\" } },\n        \"consumers\": { \"type\": \"array\", \"items\": { \"$ref\": \"#/$defs/QueueConsumer\" } }\n      }\n    },\n    \"QueueProducer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"queue\", \"binding\"],\n      \"properties\": {\n        \"queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"binding\": { \"type\": \"string\", \"minLength\": 1, \"pattern\": \"^[A-Z][A-Z0-9_]*$\" }\n      }\n    },\n    \"QueueConsumer\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"queue\"],\n      \"properties\": {\n        \"queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"max_batch_size\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 100, \"default\": 10 },\n        \"max_batch_timeout\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 60, \"default\": 5 },\n        \"max_retries\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 100, \"default\": 3 },\n        \"dead_letter_queue\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"retry_delay\": { \"type\": \"integer\", \"minimum\": 0, \"maximum\": 43200, \"default\": 0 },\n        \"max_concurrency\": { \"type\": \"integer\", \"minimum\": 1 }\n      }\n    },\n    \"SettingSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"key\", \"type\"],\n      \"properties\": {\n        \"key\": {\n          \"type\": \"string\",\n          \"pattern\": \"^[a-z][a-z0-9_]{0,63}$\",\n          \"description\": \"Setting key, unique within the plugin; the CONFIG / SECRETS property name.\"\n        },\n        \"label\": { \"type\": \"string\", \"description\": \"Operator-facing label; defaults to the key.\" },\n        \"type\": {\n          \"type\": \"string\",\n          \"enum\": [\"string\", \"bool\", \"integer\", \"enum\", \"secret\"],\n          \"description\": \"Value type. `secret` values are sealed by the host and delivered in SECRETS, never stored in the instance document.\"\n        },\n        \"values\": {\n          \"type\": \"array\",\n          \"maxItems\": 64,\n          \"description\": \"Allowed values for type = \\\"enum\\\"; forbidden otherwise.\",\n          \"items\": {\n            \"type\": \"object\",\n            \"additionalProperties\": false,\n            \"required\": [\"id\"],\n            \"properties\": {\n              \"id\": { \"type\": \"string\", \"minLength\": 1, \"maxLength\": 1024 },\n              \"label\": { \"type\": \"string\" }\n            }\n          }\n        },\n        \"default\": {\n          \"type\": [\"string\", \"boolean\", \"integer\"],\n          \"description\": \"Default applied when the instance document omits the key. Not allowed for secrets.\"\n        },\n        \"required\": { \"type\": \"boolean\", \"default\": false },\n        \"description\": { \"type\": \"string\" }\n      }\n    },\n    \"DatabaseBinding\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"binding\"],\n      \"properties\": {\n        \"binding\": {\n          \"type\": \"string\",\n          \"pattern\": \"^[A-Z][A-Z0-9_]*$\",\n          \"maxLength\": 32\n        }\n      }\n    },\n    \"NamedBinding\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    },\n    \"KvNamespace\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"binding\": { \"type\": \"string\", \"minLength\": 1 },\n        \"namespace\": {\n          \"type\": \"string\",\n          \"minLength\": 1,\n          \"maxLength\": 63,\n          \"pattern\": \"^[a-z0-9][a-z0-9._-]*$\"\n        },\n        \"max_value_bytes\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 1048576, \"default\": 65536 },\n        \"max_keys\": { \"type\": \"integer\", \"minimum\": 1, \"maximum\": 1000000, \"default\": 10000 }\n      }\n    },\n    \"WorkerdRuntime\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"compatibility_date\", \"main_module\"],\n      \"properties\": {\n        \"compatibility_date\": {\n          \"type\": \"string\",\n          \"minLength\": 10,\n          \"maxLength\": 10,\n          \"pattern\": \"^[0-9]{4}-[0-9]{2}-[0-9]{2}$\",\n          \"description\": \"Calendar YYYY-MM-DD. A date newer than the pinned workerd compatibility date 2026-10-23 warns and falls back to that pin when the isolate loads. The pattern checks shape only; checkers reject impossible days such as 2026-02-31.\"\n        },\n        \"compatibility_flags\": {\n          \"type\": \"array\",\n          \"description\": \"Exact allowlist: python_workers and disable_python_external_sdk. Both are required together when the guest is Python and forbidden otherwise. The host does not append missing flags.\",\n          \"items\": {\n            \"type\": \"string\",\n            \"enum\": [\"python_workers\", \"disable_python_external_sdk\"]\n          }\n        },\n        \"main_module\": { \"type\": \"string\", \"minLength\": 1 },\n        \"modules_dir\": { \"type\": \"string\", \"default\": \"modules\" },\n        \"entrypoint\": { \"type\": \"string\", \"default\": \"default\" },\n        \"limits\": { \"$ref\": \"#/$defs/WorkerdLimits\" }\n      }\n    },\n    \"WorkerdLimits\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"cpu_ms\": { \"type\": \"integer\", \"minimum\": 0 },\n        \"subrequests\": { \"type\": \"integer\", \"minimum\": 0 }\n      }\n    },\n    \"ModuleSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"path\": {\n          \"type\": \"string\",\n          \"description\": \"Source file relative to the plugin root. When path is omitted or empty, name is the modules-directory source.\"\n        },\n        \"type\": { \"type\": \"string\", \"default\": \"js\" }\n      }\n    },\n    \"CliSchema\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"commands\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/CliCommandSpec\" }\n        }\n      }\n    },\n    \"CliCommandSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"about\": { \"type\": \"string\" },\n        \"args\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/CliArgSpec\" }\n        }\n      }\n    },\n    \"CliArgSpec\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"name\"],\n      \"properties\": {\n        \"name\": { \"type\": \"string\" },\n        \"long\": { \"type\": \"string\" },\n        \"short\": { \"type\": \"string\", \"maxLength\": 1 },\n        \"kind\": {\n          \"type\": \"string\",\n          \"enum\": [\"string\", \"bool\", \"int\", \"path\"],\n          \"default\": \"string\"\n        },\n        \"required\": { \"type\": \"boolean\" },\n        \"default\": { \"type\": \"string\" },\n        \"about\": { \"type\": \"string\" },\n        \"positional\": { \"type\": \"boolean\" }\n      }\n    },\n    \"OidcManifest\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"properties\": {\n        \"clients\": {\n          \"type\": \"array\",\n          \"items\": { \"$ref\": \"#/$defs/OidcClient\" }\n        }\n      }\n    },\n    \"OidcClient\": {\n      \"type\": \"object\",\n      \"additionalProperties\": false,\n      \"required\": [\"client_id\", \"callback_path\", \"origin_config_key\"],\n      \"properties\": {\n        \"client_id\": { \"type\": \"string\", \"minLength\": 1 },\n        \"display_name\": { \"type\": \"string\" },\n        \"callback_path\": { \"type\": \"string\", \"pattern\": \"^/\" },\n        \"public_client\": { \"type\": \"boolean\", \"default\": true },\n        \"default_scopes\": {\n          \"type\": \"array\",\n          \"items\": { \"type\": \"string\" }\n        },\n        \"issue_refresh_token\": { \"type\": \"boolean\", \"default\": true },\n        \"origin_config_key\": { \"type\": \"string\", \"minLength\": 1 }\n      }\n    }\n  }\n}\n";
```

Embedded JSON Schema for install `plugin.toml` files (shared with language
SDK author tools and `bookclerk plugins` validation).
