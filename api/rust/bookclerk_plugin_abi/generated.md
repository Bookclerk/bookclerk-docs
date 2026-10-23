# bookclerk_plugin_abi::generated

GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.

Rust projection of the `rust-generated` section of `plugin.capnp`: owned
DTOs for every typed method payload plus `write_*` / `read_*` codecs over
the capnpc builders and readers. `$optional` scalars are `Option<T>`
(wire zero value = `None`); `{ ok, err }` reply unions are
`Result<T, PluginError>`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`EventConsumerSpec`](#eventconsumerspec) | struct | One declared event consumer: a `\[\[events.consumers\]\]` row the default entrypoint's `event(batch)` handler accepts. |
| [`PluginCapabilities`](#plugincapabilities) | struct | Typed capability declaration returned by `describe()`. The host compares it with `plugin.toml` and the operator grant; widening is rejected at spawn. |
| [`Brand`](#brand) | struct | Portal brand crossing the RPC boundary. Distinct from `plugin.toml` `logo`: `iconUrl` is the live URL or data URI the SPA renders. |
| [`ConfigOption`](#configoption) | struct | One discoverable config option group advertised for sources. |
| [`ConfigOptionValue`](#configoptionvalue) | struct | One selectable value under a `ConfigOption`. |
| [`CliSchema`](#clischema) | struct | Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`). |
| [`CliCommandSpec`](#clicommandspec) | struct | One plugin CLI command under `CliSchema`. |
| [`CliArgSpec`](#cliargspec) | struct | One CLI argument or flag under a `CliCommandSpec`. |
| [`CliArg`](#cliarg) | struct | One named argument value passed to `PluginCli.invoke`. |
| [`CliInvokeParams`](#cliinvokeparams) | struct | Params of `PluginCli.invoke`. |
| [`CliInvokeResult`](#cliinvokeresult) | struct | Result of `PluginCli.invoke`. |
| [`DatabaseAdapterConfig`](#databaseadapterconfig) | struct | Author-facing database adapter configuration carried in `DatabaseContext.adapter`. This is the generic bootstrap mechanism for third-party adapters: the operator's granted `\[database.<id>\]` table plus the scoped writable data dir. First-party host-managed adapters receive host-private connect params in `DatabaseContext.config` instead. |
| [`DiagnoseResult`](#diagnoseresult) | struct | Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`. |
| [`SourceAccount`](#sourceaccount) | struct | Source account metadata returned from login and stored by the host. |
| [`LoginParams`](#loginparams) | struct | Params of `ContentSource.login` and `ContentSource.loginStart`. Password sources fill email/password; OAuth sources use callback / external fields. There is no files-dir root or library DB path -- only `pluginDataDir`. |
| [`LoginResult`](#loginresult) | struct | Result of `ContentSource.login` / `loginComplete`: account metadata plus opaque credentials for the host to seal into `encrypted_secrets` (`provider = plugin id`). Guests never write secrets into the library DB. |
| [`LoginStartResult`](#loginstartresult) | struct | Result of `ContentSource.loginStart` (interactive OAuth). The operator opens `url`; `loginComplete` later uses `sessionId`. |
| [`LoginCompleteParams`](#logincompleteparams) | struct | Params of `ContentSource.loginComplete`. |
| [`AccountCredential`](#accountcredential) | struct | Host-sealed credentials for one account, delivered on `scan`. |
| [`ScanParams`](#scanparams) | struct | Params of `ContentSource.scan`. The host injects sealed credentials so the plugin does not need a private credential store under `pluginDataDir`. |
| [`ScanBook`](#scanbook) | struct | One library title returned by `ContentSource.scan`. The host upserts these rows and forces `source` to the plugin id. |
| [`ScanSummary`](#scansummary) | struct | Summary result of `ContentSource.scan`. |
| [`FetchOptions`](#fetchoptions) | struct | Fetch-relevant acquire knobs the host forwards so external load matches in-process. Packaging / naming knobs stay host-side. |
| [`FetchTitleParams`](#fetchtitleparams) | struct | Params of `ContentSource.fetchTitle`. The plugin writes media under `cacheDir` and returns plain (DRM-free) paths. The host injects credentials; guests must not open `library.db` or `master.key`. |
| [`PlainPart`](#plainpart) | struct | One plain audio part written under the cache directory. |
| [`ChapterMarker`](#chaptermarker) | struct | One chapter marker of a fetched title. |
| [`PlainFetch`](#plainfetch) | struct | Plain (DRM-free) fetch result. Sources always return decrypted media; DRM guests decrypt before responding. |
| [`SourceAccounts`](#sourceaccounts) | struct | Success payload of `ContentSource.listAccounts`. |
| [`SearchCatalogParams`](#searchcatalogparams) | struct | Params of `ContentSource.searchCatalog`. |
| [`ExpandCandidatesParams`](#expandcandidatesparams) | struct | Params of `ContentSource.expandCandidates`. Seed fields identify a known title; the guest returns related catalog hits. |
| [`PurchaseHintParams`](#purchasehintparams) | struct | Params of `ContentSource.purchaseHint`. At least one identity field (`productId` / `asin` / `isbn` / title+authors) should be set; guests may return `invalid_params` when none are usable. |
| [`ListDealsParams`](#listdealsparams) | struct | Params of `ContentSource.listDeals`. |
| [`CatalogDetailParams`](#catalogdetailparams) | struct | Params of `ContentSource.catalogDetail`. |
| [`CatalogHit`](#cataloghit) | struct | One catalog / candidate hit returned by `searchCatalog`, `expandCandidates`, `listDeals`, and `catalogDetail`. |
| [`CatalogHits`](#cataloghits) | struct | Success payload of the catalog list methods. |
| [`CatalogDetail`](#catalogdetail) | struct | Success payload of `ContentSource.catalogDetail`. |
| [`PurchaseHint`](#purchasehint) | struct | Purchase hint for SPA / CLI purchase deep-links. |
| [`PurchaseHintResult`](#purchasehintresult) | struct | Success payload of `ContentSource.purchaseHint`. |
| [`ScanLibraryParams`](#scanlibraryparams) | struct | Params of `RemoteLibrary.scanLibrary` (remote library sync). |
| [`AuthenticateUserParams`](#authenticateuserparams) | struct | Params of `Oidc.authenticateUser`. |
| [`ExternalUser`](#externaluser) | struct | One external user observed by an integration. The host may mint claim tickets without exposing portal details to the guest. |
| [`EventPollResult`](#eventpollresult) | struct | Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick off workflows. |
| [`ListeningProgress`](#listeningprogress) | struct | One listening-progress row. The host upserts into `listening_progress` tagged with the plugin id; plugins never open the library DB. |
| [`SyncListeningResult`](#synclisteningresult) | struct | Success payload of `RemoteLibrary.syncListening`. |
| [`PluginEvent`](#pluginevent) | struct | Guest-published domain event (`EventPublisher.publish`). The host stamps `eventId`, `source` (the plugin id), and `accountId` from the invocation; guests cannot forge either. |
| [`PublishOk`](#publishok) | struct | Success payload of `EventPublisher.publish`. |
| [`Invocation`](#invocation) | struct | Identity of one `PluginWorker.open` invocation. The host issues ids; guests echo `correlationId` / `causationId` onto published events. |
| [`QueueMessage`](#queuemessage) | struct | One message on a Workers-style queue, delivered by `QueueConsumer.queue`. |
| [`QueueBatch`](#queuebatch) | struct | One `QueueConsumer.queue` delivery: messages from one queue in send order. |
| [`QueueRetry`](#queueretry) | struct | Redeliver a queue message later. |
| [`QueueBatchOk`](#queuebatchok) | struct | Success payload of `QueueConsumer.queue`: exactly one entry per `QueueBatch.messages` entry, in order. A short list is a host-side error and the missing tail is retried. |
| [`QueueSendMessage`](#queuesendmessage) | struct | One message to send through a `\[\[queues.producers\]\]` binding. |
| [`QueueSendOk`](#queuesendok) | struct | Success payload of `QueueProducer.send` / `sendBatch`. |
| [`KvPutOptions`](#kvputoptions) | struct | Expiration and metadata for `KvNamespace.put`. |
| [`KvGetOk`](#kvgetok) | struct | Success payload of `KvNamespace.get`. |
| [`KvPutOk`](#kvputok) | struct | Success payload of `KvNamespace.put`. |
| [`KvDeleteOk`](#kvdeleteok) | struct | Success payload of `KvNamespace.delete`. |
| [`KvListOptions`](#kvlistoptions) | struct | Page request for `KvNamespace.list`. |
| [`KvListKey`](#kvlistkey) | struct | One key on a `KvNamespace.list` page. |
| [`KvListOk`](#kvlistok) | struct | Success payload of `KvNamespace.list`. |
| [`Entrypoint`](#entrypoint) | enum | Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint analogue). Each value is a capability the host calls over RPC; triggers on the default entrypoint (`event`, `job`) are declared separately. |
| [`PortalAuthMode`](#portalauthmode) | enum | Portal Accounts connect mode for storefronts. |
| [`CliArgKind`](#cliargkind) | enum | Value kind for a `CliArgSpec`. |
| [`CatalogSort`](#catalogsort) | enum | Catalog search ordering. |
| [`CatalogField`](#catalogfield) | enum | Catalog search facet restricting which field the query matches. |
| [`Abridgement`](#abridgement) | enum | Whether an edition is abridged. |
| [`QueueResult`](#queueresult) | enum | Per-message outcome inside a `QueueConsumer.queue` batch reply. |
| [`RoundTripFailure`](#roundtripfailure) | enum | Why `roundtrip_any_message` could not show `read(write(v)) == v`. |
| [`write_event_consumer_spec`](#write_event_consumer_spec) | function | Encode a `EventConsumerSpec` onto a Cap'n Proto builder. |
| [`read_event_consumer_spec`](#read_event_consumer_spec) | function | Decode a `EventConsumerSpec` from a Cap'n Proto reader. |
| [`write_plugin_capabilities`](#write_plugin_capabilities) | function | Encode a `PluginCapabilities` onto a Cap'n Proto builder. |
| [`read_plugin_capabilities`](#read_plugin_capabilities) | function | Decode a `PluginCapabilities` from a Cap'n Proto reader. |
| [`write_brand`](#write_brand) | function | Encode a `Brand` onto a Cap'n Proto builder. |
| [`read_brand`](#read_brand) | function | Decode a `Brand` from a Cap'n Proto reader. |
| [`write_config_option`](#write_config_option) | function | Encode a `ConfigOption` onto a Cap'n Proto builder. |
| [`read_config_option`](#read_config_option) | function | Decode a `ConfigOption` from a Cap'n Proto reader. |
| [`write_config_option_value`](#write_config_option_value) | function | Encode a `ConfigOptionValue` onto a Cap'n Proto builder. |
| [`read_config_option_value`](#read_config_option_value) | function | Decode a `ConfigOptionValue` from a Cap'n Proto reader. |
| [`write_cli_schema`](#write_cli_schema) | function | Encode a `CliSchema` onto a Cap'n Proto builder. |
| [`read_cli_schema`](#read_cli_schema) | function | Decode a `CliSchema` from a Cap'n Proto reader. |
| [`write_cli_command_spec`](#write_cli_command_spec) | function | Encode a `CliCommandSpec` onto a Cap'n Proto builder. |
| [`read_cli_command_spec`](#read_cli_command_spec) | function | Decode a `CliCommandSpec` from a Cap'n Proto reader. |
| [`write_cli_arg_spec`](#write_cli_arg_spec) | function | Encode a `CliArgSpec` onto a Cap'n Proto builder. |
| [`read_cli_arg_spec`](#read_cli_arg_spec) | function | Decode a `CliArgSpec` from a Cap'n Proto reader. |
| [`write_cli_arg`](#write_cli_arg) | function | Encode a `CliArg` onto a Cap'n Proto builder. |
| [`read_cli_arg`](#read_cli_arg) | function | Decode a `CliArg` from a Cap'n Proto reader. |
| [`write_cli_invoke_params`](#write_cli_invoke_params) | function | Encode a `CliInvokeParams` onto a Cap'n Proto builder. |
| [`read_cli_invoke_params`](#read_cli_invoke_params) | function | Decode a `CliInvokeParams` from a Cap'n Proto reader. |
| [`write_cli_invoke_result`](#write_cli_invoke_result) | function | Encode a `CliInvokeResult` onto a Cap'n Proto builder. |
| [`read_cli_invoke_result`](#read_cli_invoke_result) | function | Decode a `CliInvokeResult` from a Cap'n Proto reader. |
| [`write_cli_schema_reply`](#write_cli_schema_reply) | function | Encode a `CliSchemaReply` union from a typed result. |
| [`read_cli_schema_reply`](#read_cli_schema_reply) | function | Decode a `CliSchemaReply` union into a typed result. |
| [`write_cli_invoke_reply`](#write_cli_invoke_reply) | function | Encode a `CliInvokeReply` union from a typed result. |
| [`read_cli_invoke_reply`](#read_cli_invoke_reply) | function | Decode a `CliInvokeReply` union into a typed result. |
| [`write_database_adapter_config`](#write_database_adapter_config) | function | Encode a `DatabaseAdapterConfig` onto a Cap'n Proto builder. |
| [`read_database_adapter_config`](#read_database_adapter_config) | function | Decode a `DatabaseAdapterConfig` from a Cap'n Proto reader. |
| [`write_diagnose_result`](#write_diagnose_result) | function | Encode a `DiagnoseResult` onto a Cap'n Proto builder. |
| [`read_diagnose_result`](#read_diagnose_result) | function | Decode a `DiagnoseResult` from a Cap'n Proto reader. |
| [`write_diagnose_reply`](#write_diagnose_reply) | function | Encode a `DiagnoseReply` union from a typed result. |
| [`read_diagnose_reply`](#read_diagnose_reply) | function | Decode a `DiagnoseReply` union into a typed result. |
| [`write_source_account`](#write_source_account) | function | Encode a `SourceAccount` onto a Cap'n Proto builder. |
| [`read_source_account`](#read_source_account) | function | Decode a `SourceAccount` from a Cap'n Proto reader. |
| [`write_login_params`](#write_login_params) | function | Encode a `LoginParams` onto a Cap'n Proto builder. |
| [`read_login_params`](#read_login_params) | function | Decode a `LoginParams` from a Cap'n Proto reader. |
| [`write_login_result`](#write_login_result) | function | Encode a `LoginResult` onto a Cap'n Proto builder. |
| [`read_login_result`](#read_login_result) | function | Decode a `LoginResult` from a Cap'n Proto reader. |
| [`write_login_start_result`](#write_login_start_result) | function | Encode a `LoginStartResult` onto a Cap'n Proto builder. |
| [`read_login_start_result`](#read_login_start_result) | function | Decode a `LoginStartResult` from a Cap'n Proto reader. |
| [`write_login_complete_params`](#write_login_complete_params) | function | Encode a `LoginCompleteParams` onto a Cap'n Proto builder. |
| [`read_login_complete_params`](#read_login_complete_params) | function | Decode a `LoginCompleteParams` from a Cap'n Proto reader. |
| [`write_login_reply`](#write_login_reply) | function | Encode a `LoginReply` union from a typed result. |
| [`read_login_reply`](#read_login_reply) | function | Decode a `LoginReply` union into a typed result. |
| [`write_login_start_reply`](#write_login_start_reply) | function | Encode a `LoginStartReply` union from a typed result. |
| [`read_login_start_reply`](#read_login_start_reply) | function | Decode a `LoginStartReply` union into a typed result. |
| [`write_account_credential`](#write_account_credential) | function | Encode a `AccountCredential` onto a Cap'n Proto builder. |
| [`read_account_credential`](#read_account_credential) | function | Decode a `AccountCredential` from a Cap'n Proto reader. |
| [`write_scan_params`](#write_scan_params) | function | Encode a `ScanParams` onto a Cap'n Proto builder. |
| [`read_scan_params`](#read_scan_params) | function | Decode a `ScanParams` from a Cap'n Proto reader. |
| [`write_scan_book`](#write_scan_book) | function | Encode a `ScanBook` onto a Cap'n Proto builder. |
| [`read_scan_book`](#read_scan_book) | function | Decode a `ScanBook` from a Cap'n Proto reader. |
| [`write_scan_summary`](#write_scan_summary) | function | Encode a `ScanSummary` onto a Cap'n Proto builder. |
| [`read_scan_summary`](#read_scan_summary) | function | Decode a `ScanSummary` from a Cap'n Proto reader. |
| [`write_scan_reply`](#write_scan_reply) | function | Encode a `ScanReply` union from a typed result. |
| [`read_scan_reply`](#read_scan_reply) | function | Decode a `ScanReply` union into a typed result. |
| [`write_fetch_options`](#write_fetch_options) | function | Encode a `FetchOptions` onto a Cap'n Proto builder. |
| [`read_fetch_options`](#read_fetch_options) | function | Decode a `FetchOptions` from a Cap'n Proto reader. |
| [`write_fetch_title_params`](#write_fetch_title_params) | function | Encode a `FetchTitleParams` onto a Cap'n Proto builder. |
| [`read_fetch_title_params`](#read_fetch_title_params) | function | Decode a `FetchTitleParams` from a Cap'n Proto reader. |
| [`write_plain_part`](#write_plain_part) | function | Encode a `PlainPart` onto a Cap'n Proto builder. |
| [`read_plain_part`](#read_plain_part) | function | Decode a `PlainPart` from a Cap'n Proto reader. |
| [`write_chapter_marker`](#write_chapter_marker) | function | Encode a `ChapterMarker` onto a Cap'n Proto builder. |
| [`read_chapter_marker`](#read_chapter_marker) | function | Decode a `ChapterMarker` from a Cap'n Proto reader. |
| [`write_plain_fetch`](#write_plain_fetch) | function | Encode a `PlainFetch` onto a Cap'n Proto builder. |
| [`read_plain_fetch`](#read_plain_fetch) | function | Decode a `PlainFetch` from a Cap'n Proto reader. |
| [`write_fetch_title_reply`](#write_fetch_title_reply) | function | Encode a `FetchTitleReply` union from a typed result. |
| [`read_fetch_title_reply`](#read_fetch_title_reply) | function | Decode a `FetchTitleReply` union into a typed result. |
| [`write_source_accounts`](#write_source_accounts) | function | Encode a `SourceAccounts` onto a Cap'n Proto builder. |
| [`read_source_accounts`](#read_source_accounts) | function | Decode a `SourceAccounts` from a Cap'n Proto reader. |
| [`write_source_accounts_reply`](#write_source_accounts_reply) | function | Encode a `SourceAccountsReply` union from a typed result. |
| [`read_source_accounts_reply`](#read_source_accounts_reply) | function | Decode a `SourceAccountsReply` union into a typed result. |
| [`write_search_catalog_params`](#write_search_catalog_params) | function | Encode a `SearchCatalogParams` onto a Cap'n Proto builder. |
| [`read_search_catalog_params`](#read_search_catalog_params) | function | Decode a `SearchCatalogParams` from a Cap'n Proto reader. |
| [`write_expand_candidates_params`](#write_expand_candidates_params) | function | Encode a `ExpandCandidatesParams` onto a Cap'n Proto builder. |
| [`read_expand_candidates_params`](#read_expand_candidates_params) | function | Decode a `ExpandCandidatesParams` from a Cap'n Proto reader. |
| [`write_purchase_hint_params`](#write_purchase_hint_params) | function | Encode a `PurchaseHintParams` onto a Cap'n Proto builder. |
| [`read_purchase_hint_params`](#read_purchase_hint_params) | function | Decode a `PurchaseHintParams` from a Cap'n Proto reader. |
| [`write_list_deals_params`](#write_list_deals_params) | function | Encode a `ListDealsParams` onto a Cap'n Proto builder. |
| [`read_list_deals_params`](#read_list_deals_params) | function | Decode a `ListDealsParams` from a Cap'n Proto reader. |
| [`write_catalog_detail_params`](#write_catalog_detail_params) | function | Encode a `CatalogDetailParams` onto a Cap'n Proto builder. |
| [`read_catalog_detail_params`](#read_catalog_detail_params) | function | Decode a `CatalogDetailParams` from a Cap'n Proto reader. |
| [`write_catalog_hit`](#write_catalog_hit) | function | Encode a `CatalogHit` onto a Cap'n Proto builder. |
| [`read_catalog_hit`](#read_catalog_hit) | function | Decode a `CatalogHit` from a Cap'n Proto reader. |
| [`write_catalog_hits`](#write_catalog_hits) | function | Encode a `CatalogHits` onto a Cap'n Proto builder. |
| [`read_catalog_hits`](#read_catalog_hits) | function | Decode a `CatalogHits` from a Cap'n Proto reader. |
| [`write_catalog_hits_reply`](#write_catalog_hits_reply) | function | Encode a `CatalogHitsReply` union from a typed result. |
| [`read_catalog_hits_reply`](#read_catalog_hits_reply) | function | Decode a `CatalogHitsReply` union into a typed result. |
| [`write_catalog_detail`](#write_catalog_detail) | function | Encode a `CatalogDetail` onto a Cap'n Proto builder. |
| [`read_catalog_detail`](#read_catalog_detail) | function | Decode a `CatalogDetail` from a Cap'n Proto reader. |
| [`write_catalog_detail_reply`](#write_catalog_detail_reply) | function | Encode a `CatalogDetailReply` union from a typed result. |
| [`read_catalog_detail_reply`](#read_catalog_detail_reply) | function | Decode a `CatalogDetailReply` union into a typed result. |
| [`write_purchase_hint`](#write_purchase_hint) | function | Encode a `PurchaseHint` onto a Cap'n Proto builder. |
| [`read_purchase_hint`](#read_purchase_hint) | function | Decode a `PurchaseHint` from a Cap'n Proto reader. |
| [`write_purchase_hint_result`](#write_purchase_hint_result) | function | Encode a `PurchaseHintResult` onto a Cap'n Proto builder. |
| [`read_purchase_hint_result`](#read_purchase_hint_result) | function | Decode a `PurchaseHintResult` from a Cap'n Proto reader. |
| [`write_purchase_hint_reply`](#write_purchase_hint_reply) | function | Encode a `PurchaseHintReply` union from a typed result. |
| [`read_purchase_hint_reply`](#read_purchase_hint_reply) | function | Decode a `PurchaseHintReply` union into a typed result. |
| [`write_scan_library_params`](#write_scan_library_params) | function | Encode a `ScanLibraryParams` onto a Cap'n Proto builder. |
| [`read_scan_library_params`](#read_scan_library_params) | function | Decode a `ScanLibraryParams` from a Cap'n Proto reader. |
| [`write_authenticate_user_params`](#write_authenticate_user_params) | function | Encode a `AuthenticateUserParams` onto a Cap'n Proto builder. |
| [`read_authenticate_user_params`](#read_authenticate_user_params) | function | Decode a `AuthenticateUserParams` from a Cap'n Proto reader. |
| [`write_external_user`](#write_external_user) | function | Encode a `ExternalUser` onto a Cap'n Proto builder. |
| [`read_external_user`](#read_external_user) | function | Decode a `ExternalUser` from a Cap'n Proto reader. |
| [`write_external_user_reply`](#write_external_user_reply) | function | Encode a `ExternalUserReply` union from a typed result. |
| [`read_external_user_reply`](#read_external_user_reply) | function | Decode a `ExternalUserReply` union into a typed result. |
| [`write_event_poll_result`](#write_event_poll_result) | function | Encode a `EventPollResult` onto a Cap'n Proto builder. |
| [`read_event_poll_result`](#read_event_poll_result) | function | Decode a `EventPollResult` from a Cap'n Proto reader. |
| [`write_event_poll_reply`](#write_event_poll_reply) | function | Encode a `EventPollReply` union from a typed result. |
| [`read_event_poll_reply`](#read_event_poll_reply) | function | Decode a `EventPollReply` union into a typed result. |
| [`write_listening_progress`](#write_listening_progress) | function | Encode a `ListeningProgress` onto a Cap'n Proto builder. |
| [`read_listening_progress`](#read_listening_progress) | function | Decode a `ListeningProgress` from a Cap'n Proto reader. |
| [`write_sync_listening_result`](#write_sync_listening_result) | function | Encode a `SyncListeningResult` onto a Cap'n Proto builder. |
| [`read_sync_listening_result`](#read_sync_listening_result) | function | Decode a `SyncListeningResult` from a Cap'n Proto reader. |
| [`write_sync_listening_reply`](#write_sync_listening_reply) | function | Encode a `SyncListeningReply` union from a typed result. |
| [`read_sync_listening_reply`](#read_sync_listening_reply) | function | Decode a `SyncListeningReply` union into a typed result. |
| [`write_plugin_event`](#write_plugin_event) | function | Encode a `PluginEvent` onto a Cap'n Proto builder. |
| [`read_plugin_event`](#read_plugin_event) | function | Decode a `PluginEvent` from a Cap'n Proto reader. |
| [`write_publish_ok`](#write_publish_ok) | function | Encode a `PublishOk` onto a Cap'n Proto builder. |
| [`read_publish_ok`](#read_publish_ok) | function | Decode a `PublishOk` from a Cap'n Proto reader. |
| [`write_publish_reply`](#write_publish_reply) | function | Encode a `PublishReply` union from a typed result. |
| [`read_publish_reply`](#read_publish_reply) | function | Decode a `PublishReply` union into a typed result. |
| [`write_invocation`](#write_invocation) | function | Encode a `Invocation` onto a Cap'n Proto builder. |
| [`read_invocation`](#read_invocation) | function | Decode a `Invocation` from a Cap'n Proto reader. |
| [`write_queue_message`](#write_queue_message) | function | Encode a `QueueMessage` onto a Cap'n Proto builder. |
| [`read_queue_message`](#read_queue_message) | function | Decode a `QueueMessage` from a Cap'n Proto reader. |
| [`write_queue_batch`](#write_queue_batch) | function | Encode a `QueueBatch` onto a Cap'n Proto builder. |
| [`read_queue_batch`](#read_queue_batch) | function | Decode a `QueueBatch` from a Cap'n Proto reader. |
| [`write_queue_retry`](#write_queue_retry) | function | Encode a `QueueRetry` onto a Cap'n Proto builder. |
| [`read_queue_retry`](#read_queue_retry) | function | Decode a `QueueRetry` from a Cap'n Proto reader. |
| [`write_queue_result`](#write_queue_result) | function | Encode a `QueueResult` union. |
| [`read_queue_result`](#read_queue_result) | function | Decode a `QueueResult` union. |
| [`write_queue_batch_ok`](#write_queue_batch_ok) | function | Encode a `QueueBatchOk` onto a Cap'n Proto builder. |
| [`read_queue_batch_ok`](#read_queue_batch_ok) | function | Decode a `QueueBatchOk` from a Cap'n Proto reader. |
| [`write_queue_batch_reply`](#write_queue_batch_reply) | function | Encode a `QueueBatchReply` union from a typed result. |
| [`read_queue_batch_reply`](#read_queue_batch_reply) | function | Decode a `QueueBatchReply` union into a typed result. |
| [`write_queue_send_message`](#write_queue_send_message) | function | Encode a `QueueSendMessage` onto a Cap'n Proto builder. |
| [`read_queue_send_message`](#read_queue_send_message) | function | Decode a `QueueSendMessage` from a Cap'n Proto reader. |
| [`write_queue_send_ok`](#write_queue_send_ok) | function | Encode a `QueueSendOk` onto a Cap'n Proto builder. |
| [`read_queue_send_ok`](#read_queue_send_ok) | function | Decode a `QueueSendOk` from a Cap'n Proto reader. |
| [`write_queue_send_reply`](#write_queue_send_reply) | function | Encode a `QueueSendReply` union from a typed result. |
| [`read_queue_send_reply`](#read_queue_send_reply) | function | Decode a `QueueSendReply` union into a typed result. |
| [`write_kv_put_options`](#write_kv_put_options) | function | Encode a `KvPutOptions` onto a Cap'n Proto builder. |
| [`read_kv_put_options`](#read_kv_put_options) | function | Decode a `KvPutOptions` from a Cap'n Proto reader. |
| [`write_kv_get_ok`](#write_kv_get_ok) | function | Encode a `KvGetOk` onto a Cap'n Proto builder. |
| [`read_kv_get_ok`](#read_kv_get_ok) | function | Decode a `KvGetOk` from a Cap'n Proto reader. |
| [`write_kv_get_reply`](#write_kv_get_reply) | function | Encode a `KvGetReply` union from a typed result. |
| [`read_kv_get_reply`](#read_kv_get_reply) | function | Decode a `KvGetReply` union into a typed result. |
| [`write_kv_put_ok`](#write_kv_put_ok) | function | Encode a `KvPutOk` onto a Cap'n Proto builder. |
| [`read_kv_put_ok`](#read_kv_put_ok) | function | Decode a `KvPutOk` from a Cap'n Proto reader. |
| [`write_kv_delete_ok`](#write_kv_delete_ok) | function | Encode a `KvDeleteOk` onto a Cap'n Proto builder. |
| [`read_kv_delete_ok`](#read_kv_delete_ok) | function | Decode a `KvDeleteOk` from a Cap'n Proto reader. |
| [`write_kv_put_reply`](#write_kv_put_reply) | function | Encode a `KvPutReply` union from a typed result. |
| [`read_kv_put_reply`](#read_kv_put_reply) | function | Decode a `KvPutReply` union into a typed result. |
| [`write_kv_delete_reply`](#write_kv_delete_reply) | function | Encode a `KvDeleteReply` union from a typed result. |
| [`read_kv_delete_reply`](#read_kv_delete_reply) | function | Decode a `KvDeleteReply` union into a typed result. |
| [`write_kv_list_options`](#write_kv_list_options) | function | Encode a `KvListOptions` onto a Cap'n Proto builder. |
| [`read_kv_list_options`](#read_kv_list_options) | function | Decode a `KvListOptions` from a Cap'n Proto reader. |
| [`write_kv_list_key`](#write_kv_list_key) | function | Encode a `KvListKey` onto a Cap'n Proto builder. |
| [`read_kv_list_key`](#read_kv_list_key) | function | Decode a `KvListKey` from a Cap'n Proto reader. |
| [`write_kv_list_ok`](#write_kv_list_ok) | function | Encode a `KvListOk` onto a Cap'n Proto builder. |
| [`read_kv_list_ok`](#read_kv_list_ok) | function | Decode a `KvListOk` from a Cap'n Proto reader. |
| [`write_kv_list_reply`](#write_kv_list_reply) | function | Encode a `KvListReply` union from a typed result. |
| [`read_kv_list_reply`](#read_kv_list_reply) | function | Decode a `KvListReply` union into a typed result. |
| [`read_any_message`](#read_any_message) | function | Decodes `bytes` as an unpacked single-segment Cap'n Proto message whose root is the struct `selector` names, through that struct's `read_*` codec, and drops the result. |
| [`roundtrip_any_message`](#roundtrip_any_message) | function | Decodes `bytes` as the struct `selector` names (as `read_any_message` does), writes the value back through that struct's `write_*` codec, decodes the result and requires it to equal the first value. |
| [`READ_ANY_COUNT`](#read_any_count) | constant | Number of root structs `read_any_message` can decode; selectors are `0..READ_ANY_COUNT` in schema order. |

## Structs

### EventConsumerSpec

```rust
pub struct EventConsumerSpec {
    pub event_type: String,
    pub schema_versions: Vec<u32>,
    pub supports_suspend: bool,
}
```

One declared event consumer: a `\[\[events.consumers\]\]` row the default
entrypoint's `event(batch)` handler accepts.

**Fields**

- `event_type`: Versioned event type (snake_case, e.g. `book_acquired`).
- `schema_versions`: Schema versions the guest can consume; never empty.
- `supports_suspend`: Whether `EventResult.suspended` is supported for this type.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PluginCapabilities

```rust
pub struct PluginCapabilities {
    pub entrypoints: Vec<Entrypoint>,
    pub consumes: Vec<EventConsumerSpec>,
    pub produces: Vec<String>,
    pub jobs: Vec<String>,
    pub databases: Vec<String>,
    pub bindings: Vec<String>,
    pub queue_consumers: Vec<String>,
    pub queue_producers: Vec<String>,
}
```

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

**Fields**

- `entrypoints`: Named entrypoints the guest exports.
- `consumes`: Event types the default entrypoint consumes (`event(batch)` trigger).
- `produces`: Event types the guest may publish through its `EVENTS` binding.
- `jobs`: Command types the default entrypoint runs (`job(controller)` trigger).
- `databases`: Plugin-owned database binding names (`\[\[databases\]\]`).
- `bindings`: Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`,
  `WORK_FS`, `OAUTH`, `KV`, `EVENTS`, `\[\[queues.producers\]\]` names, ...).
- `queue_consumers`: Queues the default entrypoint consumes (`queue(batch)` trigger).
- `queue_producers`: Queues the guest may send to through its `\[\[queues.producers\]\]` bindings.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Brand

```rust
pub struct Brand {
    pub id: String,
    pub name: String,
    pub bg: String,
    pub fg: String,
    pub accent: String,
    pub icon_url: Option<String>,
}
```

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

**Fields**

- `id`: Brand id (often matches the plugin id); empty means "no brand".
- `name`: Display name shown next to the brand swatch.
- `bg`: Background CSS color (hex or named).
- `fg`: Foreground CSS color for text on `bg`.
- `accent`: Accent CSS color for highlights / CTAs.
- `icon_url`: Icon URL or data URI for the portal.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ConfigOption

```rust
pub struct ConfigOption {
    pub key: String,
    pub label: String,
    pub values: Vec<ConfigOptionValue>,
}
```

One discoverable config option group advertised for sources.

**Fields**

- `key`: Config key under the plugin's `config.toml` table.
- `label`: Operator-facing label for the option group.
- `values`: Allowed selectable values for this key.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ConfigOptionValue

```rust
pub struct ConfigOptionValue {
    pub id: String,
    pub label: String,
}
```

One selectable value under a `ConfigOption`.

**Fields**

- `id`: Value written to config when selected.
- `label`: Operator-facing label for this value.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliSchema

```rust
pub struct CliSchema {
    pub commands: Vec<CliCommandSpec>,
}
```

Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).

**Fields**

- `commands`: Commands exposed as `bookclerk plugins <id> <command> ...`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliCommandSpec

```rust
pub struct CliCommandSpec {
    pub name: String,
    pub about: Option<String>,
    pub args: Vec<CliArgSpec>,
}
```

One plugin CLI command under `CliSchema`.

**Fields**

- `name`: Command verb after the plugin id (for example "ping").
- `about`: Short help text for `--help`.
  `None` when absent (wire zero value).
- `args`: Argument / flag specs for this command.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliArgSpec

```rust
pub struct CliArgSpec {
    pub name: String,
    pub long: Option<String>,
    pub short: Option<String>,
    pub kind: CliArgKind,
    pub required: bool,
    pub default: Option<String>,
    pub about: Option<String>,
    pub positional: bool,
}
```

One CLI argument or flag under a `CliCommandSpec`.

**Fields**

- `name`: Internal arg name used as `CliArg.name` on invoke.
- `long`: Long flag without leading dashes (e.g. "message" -> `--message`).
  `None` when absent (wire zero value).
- `short`: Short flag character (e.g. "m" -> `-m`).
  `None` when absent (wire zero value).
- `kind`: Parsed value kind.
- `required`: When true, the host rejects invoke if the arg is missing.
- `default`: Default string form when the operator omits the arg.
  `None` when absent (wire zero value).
- `about`: Help text for this arg.
  `None` when absent (wire zero value).
- `positional`: When true, the arg is positional rather than a flagged option.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliArg

```rust
pub struct CliArg {
    pub name: String,
    pub value: String,
}
```

One named argument value passed to `PluginCli.invoke`.

**Fields**

- `name`: Arg name matching a `CliArgSpec.name`.
- `value`: String form of the value (the guest parses per `CliArgSpec.kind`).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliInvokeParams

```rust
pub struct CliInvokeParams {
    pub command: String,
    pub args: Vec<CliArg>,
}
```

Params of `PluginCli.invoke`.

**Fields**

- `command`: Command name matching a `CliCommandSpec.name`.
- `args`: Named argument values.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CliInvokeResult

```rust
pub struct CliInvokeResult {
    pub exit_code: i32,
    pub stdout: String,
    pub stderr: String,
    pub payload: ExtensibleConfig,
}
```

Result of `PluginCli.invoke`.

**Fields**

- `exit_code`: Process-style exit code (0 = success).
- `stdout`: Captured standard output text.
- `stderr`: Captured standard error text.
- `payload`: Structured payload for machine consumers; empty `mediaType` when absent.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DatabaseAdapterConfig

```rust
pub struct DatabaseAdapterConfig {
    pub plugin_data_dir: String,
    pub settings: ExtensibleConfig,
    pub binding: Option<String>,
    pub instance_id: Option<String>,
    pub open_existing: bool,
}
```

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `\[database.<id>\]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

**Fields**

- `plugin_data_dir`: Scoped writable directory for this plugin (`.../plugins/<id>/data`).
- `settings`: Granted plugin settings (operator `\[database.<id>\]` table) as
  `application/json`; `{}` when the operator configured nothing.
- `binding`: Named plugin database binding this open serves; empty for the primary
  library open. Adapters advertising `DbCapabilities.pluginDatabases` must
  serve each binding from its own isolated database.
  `None` when absent (wire zero value).
- `instance_id`: Host-issued opaque instance id for this (owner plugin, binding) pair.
  Collision-resistant and stable across re-opens. Empty for the primary
  library open. Third-party adapters must key isolated databases on this
  value rather than `binding` alone (two plugins may both declare `DB`).
  `None` when absent (wire zero value).
- `open_existing`: When true, open an existing binding unit and do not provision a missing
  one (read-only backup capture). False lets the adapter create the unit.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DiagnoseResult

```rust
pub struct DiagnoseResult {
    pub lines: Vec<String>,
}
```

Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.

**Fields**

- `lines`: Human-readable probe lines.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SourceAccount

```rust
pub struct SourceAccount {
    pub account_id: String,
    pub source: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub scan_enabled: bool,
}
```

Source account metadata returned from login and stored by the host.

**Fields**

- `account_id`: Stable account id within this source plugin.
- `source`: Source plugin id (the host forces this to the guest's install id).
- `marketplace`: Storefront marketplace / region code (for example `us`, `uk`).
- `label`: Operator-facing label.
  `None` when absent (wire zero value).
- `scan_enabled`: When true, bare / scheduled scans include this account. Explicit CLI
  `--account` bypasses this flag.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LoginParams

```rust
pub struct LoginParams {
    pub plugin_data_dir: String,
    pub marketplace: String,
    pub label: Option<String>,
    pub email: Option<String>,
    pub password: Option<String>,
    pub force: bool,
    pub callback_bind: Option<String>,
    pub callback_ipc: Option<String>,
    pub callback_public_base: Option<String>,
    pub external: bool,
    pub response_url: Option<String>,
    pub show_qr: bool,
    pub timeout_secs: Option<u64>,
    pub extra: ExtensibleConfig,
}
```

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

**Fields**

- `plugin_data_dir`: Scoped writable directory for this plugin only (`.../plugins/<id>/data`).
- `marketplace`: Marketplace / locale for the storefront; empty means the guest default.
- `label`: Operator label stored on the account row.
  `None` when absent (wire zero value).
- `email`: Account email / username for password logins; empty for pure OAuth.
  `None` when absent (wire zero value).
- `password`: Account password for password logins; never logged; empty for OAuth.
  `None` when absent (wire zero value).
- `force`: When true, overwrite an existing sealed credential for this account.
- `callback_bind`: Bind address for OAuth callback servers (`host:port`). Ignored when
  `callbackIpc` is set (host owns the TCP listener).
  `None` when absent (wire zero value).
- `callback_ipc`: Host-owned callback IPC endpoint the guest must connect to. When set
  (with `callbackPublicBase`), the guest must not bind a TCP listener.
  `None` when absent (wire zero value).
- `callback_public_base`: Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`.
  `None` when absent (wire zero value).
- `external`: When true, use external / paste-redirect OAuth instead of a local
  callback server.
- `response_url`: Pre-supplied OAuth redirect URL (paste flow).
  `None` when absent (wire zero value).
- `show_qr`: Prefer QR output when the guest supports it.
- `timeout_secs`: Seconds to wait for OAuth callback capture; guest default when 0.
  `None` when absent (wire zero value).
- `extra`: Store-specific knobs as `application/json`; guests may ignore unknowns.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LoginResult

```rust
pub struct LoginResult {
    pub account: SourceAccount,
    pub credentials: Option<Vec<u8>>,
}
```

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

**Fields**

- `account`: Account row fields for the host to upsert.
- `credentials`: Opaque credential blob the host seals; empty when login only refreshed
  metadata. Guests choose the encoding (typically JSON bytes).
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LoginStartResult

```rust
pub struct LoginStartResult {
    pub session_id: String,
    pub url: String,
}
```

Result of `ContentSource.loginStart` (interactive OAuth). The operator
opens `url`; `loginComplete` later uses `sessionId`.

**Fields**

- `session_id`: Opaque session id for `loginComplete`.
- `url`: Browser URL the operator should open to complete OAuth.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LoginCompleteParams

```rust
pub struct LoginCompleteParams {
    pub session_id: String,
}
```

Params of `ContentSource.loginComplete`.

**Fields**

- `session_id`: Session id previously returned by `loginStart`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AccountCredential

```rust
pub struct AccountCredential {
    pub account_id: String,
    pub credentials: Vec<u8>,
}
```

Host-sealed credentials for one account, delivered on `scan`.

**Fields**

- `account_id`: Account id the blob belongs to.
- `credentials`: Opaque credential bytes exactly as the guest returned them at login.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanParams

```rust
pub struct ScanParams {
    pub plugin_data_dir: String,
    pub accounts: Vec<String>,
    pub page_size: u32,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
    pub credentials: Vec<AccountCredential>,
}
```

Params of `ContentSource.scan`. The host injects sealed credentials so the
plugin does not need a private credential store under `pluginDataDir`.

**Fields**

- `plugin_data_dir`: Scoped plugin data directory.
- `accounts`: Account ids to scan; empty means all scan-enabled accounts.
- `page_size`: Storefront page size; the host always sends an explicit value.
- `import_episodes`: When true, import podcast/episode-style rows.
- `import_plus_titles`: When true, import Plus/catalog entitlement titles.
- `credentials`: Host-loaded credential blobs for the requested accounts.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanBook

```rust
pub struct ScanBook {
    pub account_id: String,
    pub product_id: String,
    pub title: String,
    pub marketplace: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub content_kind: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub subtitle: Option<String>,
}
```

One library title returned by `ContentSource.scan`. The host upserts these
rows and forces `source` to the plugin id.

**Fields**

- `account_id`: Account that owns this library entry.
- `product_id`: Storefront product / SKU id.
- `title`: Primary title string.
- `marketplace`: Marketplace / region when known.
  `None` when absent (wire zero value).
- `asin`: Amazon ASIN when the storefront exposes one.
  `None` when absent (wire zero value).
- `isbn`: ISBN when the storefront exposes one.
  `None` when absent (wire zero value).
- `authors`: Comma- or guest-formatted author list.
  `None` when absent (wire zero value).
- `narrators`: Comma- or guest-formatted narrator list.
  `None` when absent (wire zero value).
- `series`: Series name when applicable.
  `None` when absent (wire zero value).
- `series_index`: Series index / sequence label.
  `None` when absent (wire zero value).
- `content_kind`: Content classification (e.g. `book` vs `episode`).
  `None` when absent (wire zero value).
- `publisher`: Publisher name when known.
  `None` when absent (wire zero value).
- `length_minutes`: Runtime in whole minutes when known.
  `None` when absent (wire zero value).
- `subtitle`: Subtitle when distinct from `title`.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanSummary

```rust
pub struct ScanSummary {
    pub accounts: u32,
    pub books_upserted: u32,
    pub pages: u32,
    pub skipped_disabled: u32,
    pub books: Vec<ScanBook>,
}
```

Summary result of `ContentSource.scan`.

**Fields**

- `accounts`: Number of accounts touched during the scan.
- `books_upserted`: Count of titles the guest expects the host to upsert; may mirror
  `books.length`.
- `pages`: Number of storefront pages fetched.
- `skipped_disabled`: Accounts skipped because `scanEnabled` was false.
- `books`: Titles for the host to upsert. Prefer this over plugin-side DB writes.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### FetchOptions

```rust
pub struct FetchOptions {
    pub widevine: bool,
    pub xhe_aac: bool,
    pub widevine_cdm_path: Option<String>,
    pub widevine_cdm_provider: Option<String>,
    pub download_cover: bool,
    pub download_pdf: bool,
    pub cover_size: String,
    pub chapter_layout: String,
    pub strip_audible_brand_audio: bool,
    pub download_clips_bookmarks: bool,
    pub retain_aax_file: bool,
    pub download_speed_limit_kbps: u32,
    pub save_metadata_json: bool,
}
```

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

**Fields**

- `widevine`: Prefer Widevine/CENC download when the store offers it.
- `xhe_aac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm_path`: Local Widevine `.wvd` path granted to the guest.
  `None` when absent (wire zero value).
- `widevine_cdm_provider`: Remote L3 CDM provider URL; empty means the classic default, `off`
  disables remote provisioning.
  `None` when absent (wire zero value).
- `download_cover`: When true, download a cover image alongside audio.
- `download_pdf`: When true, download a companion PDF when the store exposes one.
- `cover_size`: Cover image size request (`500`, `1215`, or `native`).
- `chapter_layout`: Preferred chapter API layout when fetching (`tree` or `flat`).
- `strip_audible_brand_audio`: When true, trim Audible brand intro/outro from the remux window.
- `download_clips_bookmarks`: When true, download clips/bookmarks sidecars when offered.
- `retain_aax_file`: When true, keep the encrypted download in storage.
- `download_speed_limit_kbps`: Fetch speed cap in KB/s (`0` = unlimited).
- `save_metadata_json`: When true, persist raw catalog API JSON as `metadata.json`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### FetchTitleParams

```rust
pub struct FetchTitleParams {
    pub plugin_data_dir: String,
    pub account_id: String,
    pub title_id: String,
    pub cache_dir: String,
    pub credentials: Option<Vec<u8>>,
    pub source_config: ExtensibleConfig,
    pub fetch: FetchOptions,
}
```

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

**Fields**

- `plugin_data_dir`: Scoped plugin data directory.
- `account_id`: Account whose credentials apply.
- `title_id`: Library / storefront title id to download.
- `cache_dir`: Absolute path the guest should write media into (jail-granted TMPDIR).
- `credentials`: Host-loaded credential blob for this account; empty when unavailable.
  `None` when absent (wire zero value).
- `source_config`: Granted `\[sources.<id>\]` table as `application/json`.
- `fetch`: Fetch-relevant acquire options.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PlainPart

```rust
pub struct PlainPart {
    pub path: String,
    pub title: Option<String>,
    pub duration_ms: Option<u64>,
}
```

One plain audio part written under the cache directory.

**Fields**

- `path`: Absolute path to the part file under `cacheDir`.
- `title`: Part title (disc / chapter label).
  `None` when absent (wire zero value).
- `duration_ms`: Duration of this part in milliseconds when known.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ChapterMarker

```rust
pub struct ChapterMarker {
    pub title: String,
    pub start_ms: u64,
}
```

One chapter marker of a fetched title.

**Fields**

- `title`: Chapter title.
- `start_ms`: Chapter start offset in milliseconds from the beginning of the title.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PlainFetch

```rust
pub struct PlainFetch {
    pub parts: Vec<PlainPart>,
    pub m4b_path: Option<String>,
    pub cover_path: Option<String>,
    pub chapters: Vec<ChapterMarker>,
    pub pdf_url: Option<String>,
}
```

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

**Fields**

- `parts`: Ordered audio part files written under the cache directory.
- `m4b_path`: Single M4B path when the guest assembled one.
  `None` when absent (wire zero value).
- `cover_path`: Cover image path under the cache directory.
  `None` when absent (wire zero value).
- `chapters`: Chapter markers; empty when unknown.
- `pdf_url`: Companion PDF download URL when the store exposes one.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SourceAccounts

```rust
pub struct SourceAccounts {
    pub accounts: Vec<SourceAccount>,
}
```

Success payload of `ContentSource.listAccounts`.

**Fields**

- `accounts`: Accounts the guest knows about.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SearchCatalogParams

```rust
pub struct SearchCatalogParams {
    pub query: String,
    pub region: String,
    pub limit: u32,
    pub page: u32,
    pub sort: CatalogSort,
    pub field: CatalogField,
    pub language: Option<String>,
}
```

Params of `ContentSource.searchCatalog`.

**Fields**

- `query`: Free-text search query.
- `region`: Storefront region / marketplace code; empty means the guest default.
- `limit`: Maximum hits to return; the host always sends an explicit value.
- `page`: 1-based page for storefronts that page.
- `sort`: Sort order.
- `field`: Facet restriction.
- `language`: Preferred content language (soft-prioritize; e.g. `en`).
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ExpandCandidatesParams

```rust
pub struct ExpandCandidatesParams {
    pub source: String,
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_asin: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
    pub limit: u32,
}
```

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

**Fields**

- `source`: Source plugin id hint when expanding across storefronts.
- `product_id`: Seed storefront product id.
- `title`: Seed title text.
- `authors`: Seed authors string.
  `None` when absent (wire zero value).
- `narrators`: Seed narrators string.
  `None` when absent (wire zero value).
- `series`: Seed series name.
  `None` when absent (wire zero value).
- `series_asin`: Seed series ASIN when known.
  `None` when absent (wire zero value).
- `asin`: Seed Amazon ASIN.
  `None` when absent (wire zero value).
- `isbn`: Seed ISBN.
  `None` when absent (wire zero value).
- `region`: Storefront region / marketplace code.
- `limit`: Maximum candidates to return; the host always sends an explicit value.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PurchaseHintParams

```rust
pub struct PurchaseHintParams {
    pub product_id: Option<String>,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub region: String,
    pub with_price: bool,
}
```

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

**Fields**

- `product_id`: Storefront product id when known.
  `None` when absent (wire zero value).
- `title`: Title text for fuzzy lookup.
  `None` when absent (wire zero value).
- `authors`: Authors string for fuzzy lookup.
  `None` when absent (wire zero value).
- `asin`: Amazon ASIN when known.
  `None` when absent (wire zero value).
- `isbn`: ISBN when known.
  `None` when absent (wire zero value).
- `region`: Storefront region / marketplace code.
- `with_price`: When true, guests should include live price fields when available.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ListDealsParams

```rust
pub struct ListDealsParams {
    pub limit: Option<u32>,
}
```

Params of `ContentSource.listDeals`.

**Fields**

- `limit`: Maximum number of deals to return; guest default when 0.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogDetailParams

```rust
pub struct CatalogDetailParams {
    pub product_id: String,
    pub isbn: Option<String>,
}
```

Params of `ContentSource.catalogDetail`.

**Fields**

- `product_id`: Store product id (Libro ISBN or ISBN-slug).
- `isbn`: ISBN when it differs from `productId`.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogHit

```rust
pub struct CatalogHit {
    pub product_id: String,
    pub title: String,
    pub authors: Option<String>,
    pub narrators: Option<String>,
    pub series: Option<String>,
    pub series_index: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub url: Option<String>,
    pub cover_url: Option<String>,
    pub origin: String,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<i64>,
    pub abridgement: Abridgement,
}
```

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

**Fields**

- `product_id`: Storefront product / SKU id.
- `title`: Primary title.
- `authors`: Authors string when known.
  `None` when absent (wire zero value).
- `narrators`: Narrators string when known.
  `None` when absent (wire zero value).
- `series`: Series name when applicable.
  `None` when absent (wire zero value).
- `series_index`: Series index / sequence label.
  `None` when absent (wire zero value).
- `asin`: Amazon ASIN when known.
  `None` when absent (wire zero value).
- `isbn`: ISBN when known.
  `None` when absent (wire zero value).
- `url`: Storefront product page URL.
  `None` when absent (wire zero value).
- `cover_url`: Cover image URL.
  `None` when absent (wire zero value).
- `origin`: Hit origin label (plugin id or storefront name).
- `subtitle`: Subtitle when distinct from `title`.
  `None` when absent (wire zero value).
- `description`: Long description / blurb when fetched.
  `None` when absent (wire zero value).
- `publisher`: Publisher name when known.
  `None` when absent (wire zero value).
- `length_minutes`: Runtime in whole minutes.
  `None` when absent (wire zero value).
- `published_at`: Publication date string as provided by the storefront.
  `None` when absent (wire zero value).
- `categories`: Category / genre labels as a single string when known.
  `None` when absent (wire zero value).
- `language`: Content language code when known.
  `None` when absent (wire zero value).
- `price_cents`: Current price in minor units (cents).
  `None` when absent (wire zero value).
- `currency`: ISO currency code for `priceCents`.
  `None` when absent (wire zero value).
- `price_label`: Pre-formatted price for display.
  `None` when absent (wire zero value).
- `rating_overall`: Aggregate rating when known.
  `None` when absent (wire zero value).
- `rating_count`: Number of ratings when known.
  `None` when absent (wire zero value).
- `abridgement`: Whether the edition is abridged when the storefront says so.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### CatalogHits

```rust
pub struct CatalogHits {
    pub hits: Vec<CatalogHit>,
}
```

Success payload of the catalog list methods.

**Fields**

- `hits`: Hits in storefront order.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### CatalogDetail

```rust
pub struct CatalogDetail {
    pub found: bool,
    pub hit: CatalogHit,
}
```

Success payload of `ContentSource.catalogDetail`.

**Fields**

- `found`: False when the product is unknown to the storefront (`hit` is empty).
- `hit`: Full catalog record when `found`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### PurchaseHint

```rust
pub struct PurchaseHint {
    pub product_id: String,
    pub title: Option<String>,
    pub url: Option<String>,
    pub price_cents: Option<i64>,
    pub currency: Option<String>,
    pub price_label: Option<String>,
    pub list_price_cents: Option<i64>,
    pub list_price_label: Option<String>,
    pub member_price_cents: Option<i64>,
    pub member_price_label: Option<String>,
}
```

Purchase hint for SPA / CLI purchase deep-links.

**Fields**

- `product_id`: Storefront product id.
- `title`: Title when resolved.
  `None` when absent (wire zero value).
- `url`: Purchase or product-page URL.
  `None` when absent (wire zero value).
- `price_cents`: Current price in minor units.
  `None` when absent (wire zero value).
- `currency`: ISO currency code for price fields.
  `None` when absent (wire zero value).
- `price_label`: Pre-formatted current price.
  `None` when absent (wire zero value).
- `list_price_cents`: List / MSRP price in minor units.
  `None` when absent (wire zero value).
- `list_price_label`: Pre-formatted list price.
  `None` when absent (wire zero value).
- `member_price_cents`: Member / Plus price in minor units.
  `None` when absent (wire zero value).
- `member_price_label`: Pre-formatted member price.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PurchaseHintResult

```rust
pub struct PurchaseHintResult {
    pub found: bool,
    pub hint: PurchaseHint,
}
```

Success payload of `ContentSource.purchaseHint`.

**Fields**

- `found`: False when the guest could not resolve the title (`hint` is empty).
- `hint`: Purchase hint when `found`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanLibraryParams

```rust
pub struct ScanLibraryParams {
    pub force: bool,
}
```

Params of `RemoteLibrary.scanLibrary` (remote library sync).

**Fields**

- `force`: When true, force a full rescan even if the guest would otherwise
  incremental-sync.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AuthenticateUserParams

```rust
pub struct AuthenticateUserParams {
    pub username: String,
    pub password: String,
}
```

Params of `Oidc.authenticateUser`.

**Fields**

- `username`: Integration username / login id.
- `password`: Integration password; never logged by the host.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ExternalUser

```rust
pub struct ExternalUser {
    pub provider: String,
    pub external_user_id: String,
    pub display_name: Option<String>,
    pub access_token: Option<String>,
}
```

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

**Fields**

- `provider`: Integration provider id (often the plugin id).
- `external_user_id`: Provider-scoped user id.
- `display_name`: Display name for UI.
  `None` when absent (wire zero value).
- `access_token`: Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never
  persisted.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventPollResult

```rust
pub struct EventPollResult {
    pub users: Vec<ExternalUser>,
}
```

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

**Fields**

- `users`: Newly observed external users since the last poll.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ListeningProgress

```rust
pub struct ListeningProgress {
    pub external_user_id: String,
    pub external_item_id: String,
    pub identity_id: Option<i64>,
    pub title: Option<String>,
    pub authors: Option<String>,
    pub asin: Option<String>,
    pub isbn: Option<String>,
    pub progress: Option<f64>,
    pub current_time_seconds: Option<f64>,
    pub duration_seconds: Option<f64>,
    pub is_finished: bool,
    pub last_listened_at_unix_ms: Option<u64>,
}
```

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

**Fields**

- `external_user_id`: Provider-scoped user id.
- `external_item_id`: Provider-scoped item / library id.
- `identity_id`: Bookclerk identity row id when already linked.
  `None` when absent (wire zero value).
- `title`: Title text when known.
  `None` when absent (wire zero value).
- `authors`: Authors string when known.
  `None` when absent (wire zero value).
- `asin`: Amazon ASIN when known.
  `None` when absent (wire zero value).
- `isbn`: ISBN when known.
  `None` when absent (wire zero value).
- `progress`: Fractional progress in `0.0..=1.0` when the provider reports it.
  `None` when absent (wire zero value).
- `current_time_seconds`: Current playback position in seconds.
  `None` when absent (wire zero value).
- `duration_seconds`: Total duration in seconds when known.
  `None` when absent (wire zero value).
- `is_finished`: When true, the provider marks the item finished.
- `last_listened_at_unix_ms`: Last listen timestamp as unix milliseconds (UTC).
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### SyncListeningResult

```rust
pub struct SyncListeningResult {
    pub items: Vec<ListeningProgress>,
}
```

Success payload of `RemoteLibrary.syncListening`.

**Fields**

- `items`: Progress snapshots to upsert.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### PluginEvent

```rust
pub struct PluginEvent {
    pub event_type: String,
    pub schema_version: u32,
    pub deduplication_key: String,
    pub payload: Vec<u8>,
    pub occurred_at_unix_ms: u64,
    pub correlation_id: String,
    pub causation_id: String,
}
```

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

**Fields**

- `event_type`: Snake_case event type; must be listed in `\[\[events.producers\]\]`.
- `schema_version`: Schema version of `payload`, owned by the event type.
- `deduplication_key`: Producer idempotency key, unique per (account, source, eventType) in the
  outbox; a repeat returns `PublishOk.duplicate = true` with the earlier id.
  Empty publishes unconditionally.
- `payload`: Encoded event payload; at most `maxEventPayloadBytes`.
- `occurred_at_unix_ms`: When the producer observed the fact; zero means "now" on the host clock.
- `correlation_id`: Trace correlation id; empty inherits `Invocation.correlationId`.
- `causation_id`: Id of the event or command that caused this one; empty inherits
  `Invocation.causationId`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PublishOk

```rust
pub struct PublishOk {
    pub event_id: String,
    pub duplicate: bool,
}
```

Success payload of `EventPublisher.publish`.

**Fields**

- `event_id`: Outbox event id (new or the earlier row when `duplicate`).
- `duplicate`: True when `deduplicationKey` matched an existing outbox row.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Invocation

```rust
pub struct Invocation {
    pub id: String,
    pub account_id: String,
    pub deadline_unix_ms: u64,
    pub correlation_id: String,
    pub causation_id: String,
}
```

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

**Fields**

- `id`: Unique host-issued invocation id.
- `account_id`: Account scope; empty for operator / host-wide invocations.
- `deadline_unix_ms`: UTC Unix milliseconds; zero when the invocation has no deadline. The host
  fence is authoritative.
- `correlation_id`: Trace correlation id; empty when none.
- `causation_id`: Id of the event or command that caused this invocation; empty when none.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueMessage

```rust
pub struct QueueMessage {
    pub id: String,
    pub timestamp_unix_ms: u64,
    pub body: Vec<u8>,
    pub content_type: String,
    pub attempts: u32,
    pub source: String,
}
```

One message on a Workers-style queue, delivered by `QueueConsumer.queue`.

**Fields**

- `id`: Outbox row id; stable across redeliveries.
- `timestamp_unix_ms`: When the producer sent the message (UTC Unix milliseconds).
- `body`: Encoded body; at most `maxEventPayloadBytes`.
- `content_type`: Body encoding: `json`, `text`, or `bytes`.
- `attempts`: Delivery counter, starting at 1.
- `source`: Producer plugin id; empty when the host sent it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueBatch

```rust
pub struct QueueBatch {
    pub queue: String,
    pub messages: Vec<QueueMessage>,
}
```

One `QueueConsumer.queue` delivery: messages from one queue in send order.

**Fields**

- `queue`: Queue the messages were sent to.
- `messages`: Messages in delivery order; the reply carries one `QueueResult` per entry.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueRetry

```rust
pub struct QueueRetry {
    pub delay_seconds: u32,
    pub reason: String,
}
```

Redeliver a queue message later.

**Fields**

- `delay_seconds`: Seconds before redelivery; zero applies the consumer's `retry_delay`.
- `reason`: Short human reason.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueBatchOk

```rust
pub struct QueueBatchOk {
    pub results: Vec<QueueResult>,
}
```

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

**Fields**

- `results`: Per-message outcomes in delivery order.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueSendMessage

```rust
pub struct QueueSendMessage {
    pub body: Vec<u8>,
    pub content_type: String,
    pub delay_seconds: u32,
}
```

One message to send through a `\[\[queues.producers\]\]` binding.

**Fields**

- `body`: Encoded body; at most `maxEventPayloadBytes`.
- `content_type`: Body encoding: `json` (default when empty), `text`, or `bytes`.
- `delay_seconds`: Seconds before the message may be delivered; zero means now.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QueueSendOk

```rust
pub struct QueueSendOk {
    pub message_ids: Vec<String>,
}
```

Success payload of `QueueProducer.send` / `sendBatch`.

**Fields**

- `message_ids`: Outbox row ids in send order.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvPutOptions

```rust
pub struct KvPutOptions {
    pub expiration_unix_ms: u64,
    pub expiration_ttl_seconds: u64,
    pub metadata: String,
}
```

Expiration and metadata for `KvNamespace.put`.

**Fields**

- `expiration_unix_ms`: Absolute expiry (UTC Unix milliseconds); zero means none.
- `expiration_ttl_seconds`: Relative expiry in seconds from the write; zero means none. Ignored when
  `expirationUnixMs` is set.
- `metadata`: Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvGetOk

```rust
pub struct KvGetOk {
    pub found: bool,
    pub value: Vec<u8>,
    pub metadata: String,
    pub expiration_unix_ms: u64,
}
```

Success payload of `KvNamespace.get`.

**Fields**

- `found`: True when the key exists and has not expired.
- `value`: Stored value; empty when not found.
- `metadata`: Stored metadata JSON text; empty when none.
- `expiration_unix_ms`: Absolute expiry (UTC Unix milliseconds); zero means none.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvPutOk

```rust
pub struct KvPutOk {
    pub expiration_unix_ms: u64,
}
```

Success payload of `KvNamespace.put`.

**Fields**

- `expiration_unix_ms`: Effective absolute expiry (UTC Unix milliseconds); zero means none.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvDeleteOk

```rust
pub struct KvDeleteOk {
    pub existed: bool,
}
```

Success payload of `KvNamespace.delete`.

**Fields**

- `existed`: True when a live value existed before the delete.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvListOptions

```rust
pub struct KvListOptions {
    pub prefix: String,
    pub cursor: String,
    pub limit: u32,
}
```

Page request for `KvNamespace.list`.

**Fields**

- `prefix`: Only keys starting with this prefix; empty means all.
- `cursor`: Opaque cursor from a previous page; empty means the first page.
- `limit`: Most keys to return; zero means the host default, at most `maxListPage`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvListKey

```rust
pub struct KvListKey {
    pub key: String,
    pub metadata: String,
    pub expiration_unix_ms: u64,
}
```

One key on a `KvNamespace.list` page.

**Fields**

- `key`: Key name.
- `metadata`: Stored metadata JSON text; empty when none.
- `expiration_unix_ms`: Absolute expiry (UTC Unix milliseconds); zero means none.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### KvListOk

```rust
pub struct KvListOk {
    pub keys: Vec<KvListKey>,
    pub list_complete: bool,
    pub cursor: String,
}
```

Success payload of `KvNamespace.list`.

**Fields**

- `keys`: Keys in lexical order.
- `list_complete`: True when no further page exists.
- `cursor`: Cursor for the next page; empty when `listComplete`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

## Enums

### Entrypoint

```rust
pub enum Entrypoint {
    Storefront,
    Storage,
    DatabaseAdapter,
    RemoteLibrary,
    Cli,
    Oidc,
}
```

Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
analogue). Each value is a capability the host calls over RPC; triggers on
the default entrypoint (`event`, `job`) are declared separately.

**Variants**

- `Storefront`: Storefront: login, scan, fetch, catalog search (`ContentSource`).
- `Storage`: Object storage destination (`Destination`).
- `DatabaseAdapter`: Library database adapter (`Database`).
- `RemoteLibrary`: Remote-library lifecycle: start/stop, scanLibrary, syncListening,
  pollEvents (`RemoteLibrary`).
- `Cli`: Guest CLI (`PluginCli.describe` / `PluginCli.invoke`).
- `Oidc`: OIDC bridge: relying-party client templates and `authenticateUser`.

#### Entrypoint::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### Entrypoint::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<Entrypoint>`, `Hash`, `PartialEq`, `Serialize`

### PortalAuthMode

```rust
pub enum PortalAuthMode {
    Unspecified,
    Password,
    Oauth,
}
```

Portal Accounts connect mode for storefronts.

**Variants**

- `Unspecified`: Guest did not declare a mode; the host assumes password login.
- `Password`: Email / password login through the Accounts UI.
- `Oauth`: Browser OAuth through `loginStart` / `loginComplete`.

#### PortalAuthMode::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### PortalAuthMode::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<PortalAuthMode>`, `Hash`, `PartialEq`, `Serialize`

### CliArgKind

```rust
pub enum CliArgKind {
    String,
    Bool,
    Int,
    Path,
}
```

Value kind for a `CliArgSpec`.

**Variants**

- `String`: Free-form string argument (default).
- `Bool`: Boolean flag ("true" / "false").
- `Int`: Integer argument.
- `Path`: Filesystem path argument.

#### CliArgKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### CliArgKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CliArgKind>`, `Hash`, `PartialEq`, `Serialize`

### CatalogSort

```rust
pub enum CatalogSort {
    Relevance,
    Popularity,
    Rating,
    Title,
    Author,
}
```

Catalog search ordering.

**Variants**

- `Relevance`: Storefront relevance ranking (default).
- `Popularity`: Most popular first.
- `Rating`: Highest rated first.
- `Title`: Alphabetical by title.
- `Author`: Alphabetical by author.

#### CatalogSort::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### CatalogSort::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CatalogSort>`, `Hash`, `PartialEq`, `Serialize`

### CatalogField

```rust
pub enum CatalogField {
    Any,
    Author,
    Narrator,
    Series,
    Genre,
}
```

Catalog search facet restricting which field the query matches.

**Variants**

- `Any`: Match any field.
- `Author`: Match author names.
- `Narrator`: Match narrator names.
- `Series`: Match series names.
- `Genre`: Match genre / category labels.

#### CatalogField::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### CatalogField::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CatalogField>`, `Hash`, `PartialEq`, `Serialize`

### Abridgement

```rust
pub enum Abridgement {
    Unknown,
    Unabridged,
    Abridged,
}
```

Whether an edition is abridged.

**Variants**

- `Unknown`: The storefront did not say.
- `Unabridged`: Unabridged edition.
- `Abridged`: Abridged edition.

#### Abridgement::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### Abridgement::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<Abridgement>`, `Hash`, `PartialEq`, `Serialize`

### QueueResult

```rust
pub enum QueueResult {
    Ack,
    Retry(QueueRetry),
}
```

Per-message outcome inside a `QueueConsumer.queue` batch reply.

**Variants**

- `Ack`: Handled; the host marks the message delivered.
- `Retry`: Redeliver later.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### RoundTripFailure

```rust
pub enum RoundTripFailure {
    Rejected(Error),
    Codec(String),
}
```

Why [`roundtrip_any_message`](#roundtrip_any_message) could not show `read(write(v)) == v`.

**Variants**

- `Rejected`: The input bytes did not decode; expected for arbitrary input and
  not a codec defect.
- `Codec`: A value that did decode failed to re-encode, to decode again, or
  came back different: a defect in a generated codec pair.

Implements: `Debug`, `From<Error>`

## Functions

### write_event_consumer_spec

```rust
pub fn write_event_consumer_spec(b: Builder<'_>, v: &EventConsumerSpec) -> Result<()>
```

Encode a [`EventConsumerSpec`](#eventconsumerspec) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_event_consumer_spec

```rust
pub fn read_event_consumer_spec(r: Reader<'_>) -> Result<EventConsumerSpec>
```

Decode a [`EventConsumerSpec`](#eventconsumerspec) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_plugin_capabilities

```rust
pub fn write_plugin_capabilities(b: Builder<'_>, v: &PluginCapabilities) -> Result<()>
```

Encode a [`PluginCapabilities`](#plugincapabilities) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_plugin_capabilities

```rust
pub fn read_plugin_capabilities(r: Reader<'_>) -> Result<PluginCapabilities>
```

Decode a [`PluginCapabilities`](#plugincapabilities) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_brand

```rust
pub fn write_brand(b: Builder<'_>, v: &Brand) -> Result<()>
```

Encode a [`Brand`](#brand) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_brand

```rust
pub fn read_brand(r: Reader<'_>) -> Result<Brand>
```

Decode a [`Brand`](#brand) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_config_option

```rust
pub fn write_config_option(b: Builder<'_>, v: &ConfigOption) -> Result<()>
```

Encode a [`ConfigOption`](#configoption) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_config_option

```rust
pub fn read_config_option(r: Reader<'_>) -> Result<ConfigOption>
```

Decode a [`ConfigOption`](#configoption) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_config_option_value

```rust
pub fn write_config_option_value(b: Builder<'_>, v: &ConfigOptionValue) -> Result<()>
```

Encode a [`ConfigOptionValue`](#configoptionvalue) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_config_option_value

```rust
pub fn read_config_option_value(r: Reader<'_>) -> Result<ConfigOptionValue>
```

Decode a [`ConfigOptionValue`](#configoptionvalue) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_schema

```rust
pub fn write_cli_schema(b: Builder<'_>, v: &CliSchema) -> Result<()>
```

Encode a [`CliSchema`](#clischema) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_schema

```rust
pub fn read_cli_schema(r: Reader<'_>) -> Result<CliSchema>
```

Decode a [`CliSchema`](#clischema) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_command_spec

```rust
pub fn write_cli_command_spec(b: Builder<'_>, v: &CliCommandSpec) -> Result<()>
```

Encode a [`CliCommandSpec`](#clicommandspec) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_command_spec

```rust
pub fn read_cli_command_spec(r: Reader<'_>) -> Result<CliCommandSpec>
```

Decode a [`CliCommandSpec`](#clicommandspec) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_arg_spec

```rust
pub fn write_cli_arg_spec(b: Builder<'_>, v: &CliArgSpec) -> Result<()>
```

Encode a [`CliArgSpec`](#cliargspec) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_arg_spec

```rust
pub fn read_cli_arg_spec(r: Reader<'_>) -> Result<CliArgSpec>
```

Decode a [`CliArgSpec`](#cliargspec) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_arg

```rust
pub fn write_cli_arg(b: Builder<'_>, v: &CliArg) -> Result<()>
```

Encode a [`CliArg`](#cliarg) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_arg

```rust
pub fn read_cli_arg(r: Reader<'_>) -> Result<CliArg>
```

Decode a [`CliArg`](#cliarg) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_invoke_params

```rust
pub fn write_cli_invoke_params(b: Builder<'_>, v: &CliInvokeParams) -> Result<()>
```

Encode a [`CliInvokeParams`](#cliinvokeparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_invoke_params

```rust
pub fn read_cli_invoke_params(r: Reader<'_>) -> Result<CliInvokeParams>
```

Decode a [`CliInvokeParams`](#cliinvokeparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_invoke_result

```rust
pub fn write_cli_invoke_result(b: Builder<'_>, v: &CliInvokeResult) -> Result<()>
```

Encode a [`CliInvokeResult`](#cliinvokeresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cli_invoke_result

```rust
pub fn read_cli_invoke_result(r: Reader<'_>) -> Result<CliInvokeResult>
```

Decode a [`CliInvokeResult`](#cliinvokeresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cli_schema_reply

```rust
pub fn write_cli_schema_reply(
    b: Builder<'_>,
    v: &Result<CliSchema, PluginError>,
) -> Result<()>
```

Encode a `CliSchemaReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_cli_schema_reply

```rust
pub fn read_cli_schema_reply(r: Reader<'_>) -> Result<Result<CliSchema, PluginError>>
```

Decode a `CliSchemaReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_cli_invoke_reply

```rust
pub fn write_cli_invoke_reply(
    b: Builder<'_>,
    v: &Result<CliInvokeResult, PluginError>,
) -> Result<()>
```

Encode a `CliInvokeReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_cli_invoke_reply

```rust
pub fn read_cli_invoke_reply(
    r: Reader<'_>,
) -> Result<Result<CliInvokeResult, PluginError>>
```

Decode a `CliInvokeReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_database_adapter_config

```rust
pub fn write_database_adapter_config(
    b: Builder<'_>,
    v: &DatabaseAdapterConfig,
) -> Result<()>
```

Encode a [`DatabaseAdapterConfig`](#databaseadapterconfig) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_database_adapter_config

```rust
pub fn read_database_adapter_config(r: Reader<'_>) -> Result<DatabaseAdapterConfig>
```

Decode a [`DatabaseAdapterConfig`](#databaseadapterconfig) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_diagnose_result

```rust
pub fn write_diagnose_result(b: Builder<'_>, v: &DiagnoseResult) -> Result<()>
```

Encode a [`DiagnoseResult`](#diagnoseresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_diagnose_result

```rust
pub fn read_diagnose_result(r: Reader<'_>) -> Result<DiagnoseResult>
```

Decode a [`DiagnoseResult`](#diagnoseresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_diagnose_reply

```rust
pub fn write_diagnose_reply(
    b: Builder<'_>,
    v: &Result<DiagnoseResult, PluginError>,
) -> Result<()>
```

Encode a `DiagnoseReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_diagnose_reply

```rust
pub fn read_diagnose_reply(r: Reader<'_>) -> Result<Result<DiagnoseResult, PluginError>>
```

Decode a `DiagnoseReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_source_account

```rust
pub fn write_source_account(b: Builder<'_>, v: &SourceAccount) -> Result<()>
```

Encode a [`SourceAccount`](#sourceaccount) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_source_account

```rust
pub fn read_source_account(r: Reader<'_>) -> Result<SourceAccount>
```

Decode a [`SourceAccount`](#sourceaccount) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_login_params

```rust
pub fn write_login_params(b: Builder<'_>, v: &LoginParams) -> Result<()>
```

Encode a [`LoginParams`](#loginparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_login_params

```rust
pub fn read_login_params(r: Reader<'_>) -> Result<LoginParams>
```

Decode a [`LoginParams`](#loginparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_login_result

```rust
pub fn write_login_result(b: Builder<'_>, v: &LoginResult) -> Result<()>
```

Encode a [`LoginResult`](#loginresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_login_result

```rust
pub fn read_login_result(r: Reader<'_>) -> Result<LoginResult>
```

Decode a [`LoginResult`](#loginresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_login_start_result

```rust
pub fn write_login_start_result(b: Builder<'_>, v: &LoginStartResult) -> Result<()>
```

Encode a [`LoginStartResult`](#loginstartresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_login_start_result

```rust
pub fn read_login_start_result(r: Reader<'_>) -> Result<LoginStartResult>
```

Decode a [`LoginStartResult`](#loginstartresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_login_complete_params

```rust
pub fn write_login_complete_params(b: Builder<'_>, v: &LoginCompleteParams) -> Result<()>
```

Encode a [`LoginCompleteParams`](#logincompleteparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_login_complete_params

```rust
pub fn read_login_complete_params(r: Reader<'_>) -> Result<LoginCompleteParams>
```

Decode a [`LoginCompleteParams`](#logincompleteparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_login_reply

```rust
pub fn write_login_reply(
    b: Builder<'_>,
    v: &Result<LoginResult, PluginError>,
) -> Result<()>
```

Encode a `LoginReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_login_reply

```rust
pub fn read_login_reply(r: Reader<'_>) -> Result<Result<LoginResult, PluginError>>
```

Decode a `LoginReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_login_start_reply

```rust
pub fn write_login_start_reply(
    b: Builder<'_>,
    v: &Result<LoginStartResult, PluginError>,
) -> Result<()>
```

Encode a `LoginStartReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_login_start_reply

```rust
pub fn read_login_start_reply(
    r: Reader<'_>,
) -> Result<Result<LoginStartResult, PluginError>>
```

Decode a `LoginStartReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_account_credential

```rust
pub fn write_account_credential(b: Builder<'_>, v: &AccountCredential) -> Result<()>
```

Encode a [`AccountCredential`](#accountcredential) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_account_credential

```rust
pub fn read_account_credential(r: Reader<'_>) -> Result<AccountCredential>
```

Decode a [`AccountCredential`](#accountcredential) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_scan_params

```rust
pub fn write_scan_params(b: Builder<'_>, v: &ScanParams) -> Result<()>
```

Encode a [`ScanParams`](#scanparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scan_params

```rust
pub fn read_scan_params(r: Reader<'_>) -> Result<ScanParams>
```

Decode a [`ScanParams`](#scanparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_scan_book

```rust
pub fn write_scan_book(b: Builder<'_>, v: &ScanBook) -> Result<()>
```

Encode a [`ScanBook`](#scanbook) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scan_book

```rust
pub fn read_scan_book(r: Reader<'_>) -> Result<ScanBook>
```

Decode a [`ScanBook`](#scanbook) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_scan_summary

```rust
pub fn write_scan_summary(b: Builder<'_>, v: &ScanSummary) -> Result<()>
```

Encode a [`ScanSummary`](#scansummary) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scan_summary

```rust
pub fn read_scan_summary(r: Reader<'_>) -> Result<ScanSummary>
```

Decode a [`ScanSummary`](#scansummary) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_scan_reply

```rust
pub fn write_scan_reply(
    b: Builder<'_>,
    v: &Result<ScanSummary, PluginError>,
) -> Result<()>
```

Encode a `ScanReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_scan_reply

```rust
pub fn read_scan_reply(r: Reader<'_>) -> Result<Result<ScanSummary, PluginError>>
```

Decode a `ScanReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_fetch_options

```rust
pub fn write_fetch_options(b: Builder<'_>, v: &FetchOptions) -> Result<()>
```

Encode a [`FetchOptions`](#fetchoptions) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_fetch_options

```rust
pub fn read_fetch_options(r: Reader<'_>) -> Result<FetchOptions>
```

Decode a [`FetchOptions`](#fetchoptions) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_fetch_title_params

```rust
pub fn write_fetch_title_params(b: Builder<'_>, v: &FetchTitleParams) -> Result<()>
```

Encode a [`FetchTitleParams`](#fetchtitleparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_fetch_title_params

```rust
pub fn read_fetch_title_params(r: Reader<'_>) -> Result<FetchTitleParams>
```

Decode a [`FetchTitleParams`](#fetchtitleparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_plain_part

```rust
pub fn write_plain_part(b: Builder<'_>, v: &PlainPart) -> Result<()>
```

Encode a [`PlainPart`](#plainpart) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_plain_part

```rust
pub fn read_plain_part(r: Reader<'_>) -> Result<PlainPart>
```

Decode a [`PlainPart`](#plainpart) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_chapter_marker

```rust
pub fn write_chapter_marker(b: Builder<'_>, v: &ChapterMarker) -> Result<()>
```

Encode a [`ChapterMarker`](#chaptermarker) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_chapter_marker

```rust
pub fn read_chapter_marker(r: Reader<'_>) -> Result<ChapterMarker>
```

Decode a [`ChapterMarker`](#chaptermarker) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_plain_fetch

```rust
pub fn write_plain_fetch(b: Builder<'_>, v: &PlainFetch) -> Result<()>
```

Encode a [`PlainFetch`](#plainfetch) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_plain_fetch

```rust
pub fn read_plain_fetch(r: Reader<'_>) -> Result<PlainFetch>
```

Decode a [`PlainFetch`](#plainfetch) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_fetch_title_reply

```rust
pub fn write_fetch_title_reply(
    b: Builder<'_>,
    v: &Result<PlainFetch, PluginError>,
) -> Result<()>
```

Encode a `FetchTitleReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_fetch_title_reply

```rust
pub fn read_fetch_title_reply(r: Reader<'_>) -> Result<Result<PlainFetch, PluginError>>
```

Decode a `FetchTitleReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_source_accounts

```rust
pub fn write_source_accounts(b: Builder<'_>, v: &SourceAccounts) -> Result<()>
```

Encode a [`SourceAccounts`](#sourceaccounts) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_source_accounts

```rust
pub fn read_source_accounts(r: Reader<'_>) -> Result<SourceAccounts>
```

Decode a [`SourceAccounts`](#sourceaccounts) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_source_accounts_reply

```rust
pub fn write_source_accounts_reply(
    b: Builder<'_>,
    v: &Result<SourceAccounts, PluginError>,
) -> Result<()>
```

Encode a `SourceAccountsReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_source_accounts_reply

```rust
pub fn read_source_accounts_reply(
    r: Reader<'_>,
) -> Result<Result<SourceAccounts, PluginError>>
```

Decode a `SourceAccountsReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_search_catalog_params

```rust
pub fn write_search_catalog_params(b: Builder<'_>, v: &SearchCatalogParams) -> Result<()>
```

Encode a [`SearchCatalogParams`](#searchcatalogparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_search_catalog_params

```rust
pub fn read_search_catalog_params(r: Reader<'_>) -> Result<SearchCatalogParams>
```

Decode a [`SearchCatalogParams`](#searchcatalogparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_expand_candidates_params

```rust
pub fn write_expand_candidates_params(
    b: Builder<'_>,
    v: &ExpandCandidatesParams,
) -> Result<()>
```

Encode a [`ExpandCandidatesParams`](#expandcandidatesparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_expand_candidates_params

```rust
pub fn read_expand_candidates_params(r: Reader<'_>) -> Result<ExpandCandidatesParams>
```

Decode a [`ExpandCandidatesParams`](#expandcandidatesparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_purchase_hint_params

```rust
pub fn write_purchase_hint_params(b: Builder<'_>, v: &PurchaseHintParams) -> Result<()>
```

Encode a [`PurchaseHintParams`](#purchasehintparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_purchase_hint_params

```rust
pub fn read_purchase_hint_params(r: Reader<'_>) -> Result<PurchaseHintParams>
```

Decode a [`PurchaseHintParams`](#purchasehintparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_list_deals_params

```rust
pub fn write_list_deals_params(b: Builder<'_>, v: &ListDealsParams) -> Result<()>
```

Encode a [`ListDealsParams`](#listdealsparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_list_deals_params

```rust
pub fn read_list_deals_params(r: Reader<'_>) -> Result<ListDealsParams>
```

Decode a [`ListDealsParams`](#listdealsparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_catalog_detail_params

```rust
pub fn write_catalog_detail_params(b: Builder<'_>, v: &CatalogDetailParams) -> Result<()>
```

Encode a [`CatalogDetailParams`](#catalogdetailparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_catalog_detail_params

```rust
pub fn read_catalog_detail_params(r: Reader<'_>) -> Result<CatalogDetailParams>
```

Decode a [`CatalogDetailParams`](#catalogdetailparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_catalog_hit

```rust
pub fn write_catalog_hit(b: Builder<'_>, v: &CatalogHit) -> Result<()>
```

Encode a [`CatalogHit`](#cataloghit) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_catalog_hit

```rust
pub fn read_catalog_hit(r: Reader<'_>) -> Result<CatalogHit>
```

Decode a [`CatalogHit`](#cataloghit) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_catalog_hits

```rust
pub fn write_catalog_hits(b: Builder<'_>, v: &CatalogHits) -> Result<()>
```

Encode a [`CatalogHits`](#cataloghits) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_catalog_hits

```rust
pub fn read_catalog_hits(r: Reader<'_>) -> Result<CatalogHits>
```

Decode a [`CatalogHits`](#cataloghits) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_catalog_hits_reply

```rust
pub fn write_catalog_hits_reply(
    b: Builder<'_>,
    v: &Result<CatalogHits, PluginError>,
) -> Result<()>
```

Encode a `CatalogHitsReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_catalog_hits_reply

```rust
pub fn read_catalog_hits_reply(r: Reader<'_>) -> Result<Result<CatalogHits, PluginError>>
```

Decode a `CatalogHitsReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_catalog_detail

```rust
pub fn write_catalog_detail(b: Builder<'_>, v: &CatalogDetail) -> Result<()>
```

Encode a [`CatalogDetail`](#catalogdetail) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_catalog_detail

```rust
pub fn read_catalog_detail(r: Reader<'_>) -> Result<CatalogDetail>
```

Decode a [`CatalogDetail`](#catalogdetail) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_catalog_detail_reply

```rust
pub fn write_catalog_detail_reply(
    b: Builder<'_>,
    v: &Result<CatalogDetail, PluginError>,
) -> Result<()>
```

Encode a `CatalogDetailReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_catalog_detail_reply

```rust
pub fn read_catalog_detail_reply(
    r: Reader<'_>,
) -> Result<Result<CatalogDetail, PluginError>>
```

Decode a `CatalogDetailReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_purchase_hint

```rust
pub fn write_purchase_hint(b: Builder<'_>, v: &PurchaseHint) -> Result<()>
```

Encode a [`PurchaseHint`](#purchasehint) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_purchase_hint

```rust
pub fn read_purchase_hint(r: Reader<'_>) -> Result<PurchaseHint>
```

Decode a [`PurchaseHint`](#purchasehint) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_purchase_hint_result

```rust
pub fn write_purchase_hint_result(b: Builder<'_>, v: &PurchaseHintResult) -> Result<()>
```

Encode a [`PurchaseHintResult`](#purchasehintresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_purchase_hint_result

```rust
pub fn read_purchase_hint_result(r: Reader<'_>) -> Result<PurchaseHintResult>
```

Decode a [`PurchaseHintResult`](#purchasehintresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_purchase_hint_reply

```rust
pub fn write_purchase_hint_reply(
    b: Builder<'_>,
    v: &Result<PurchaseHintResult, PluginError>,
) -> Result<()>
```

Encode a `PurchaseHintReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_purchase_hint_reply

```rust
pub fn read_purchase_hint_reply(
    r: Reader<'_>,
) -> Result<Result<PurchaseHintResult, PluginError>>
```

Decode a `PurchaseHintReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_scan_library_params

```rust
pub fn write_scan_library_params(b: Builder<'_>, v: &ScanLibraryParams) -> Result<()>
```

Encode a [`ScanLibraryParams`](#scanlibraryparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scan_library_params

```rust
pub fn read_scan_library_params(r: Reader<'_>) -> Result<ScanLibraryParams>
```

Decode a [`ScanLibraryParams`](#scanlibraryparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_authenticate_user_params

```rust
pub fn write_authenticate_user_params(
    b: Builder<'_>,
    v: &AuthenticateUserParams,
) -> Result<()>
```

Encode a [`AuthenticateUserParams`](#authenticateuserparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_authenticate_user_params

```rust
pub fn read_authenticate_user_params(r: Reader<'_>) -> Result<AuthenticateUserParams>
```

Decode a [`AuthenticateUserParams`](#authenticateuserparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_external_user

```rust
pub fn write_external_user(b: Builder<'_>, v: &ExternalUser) -> Result<()>
```

Encode a [`ExternalUser`](#externaluser) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_external_user

```rust
pub fn read_external_user(r: Reader<'_>) -> Result<ExternalUser>
```

Decode a [`ExternalUser`](#externaluser) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_external_user_reply

```rust
pub fn write_external_user_reply(
    b: Builder<'_>,
    v: &Result<ExternalUser, PluginError>,
) -> Result<()>
```

Encode a `ExternalUserReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_external_user_reply

```rust
pub fn read_external_user_reply(
    r: Reader<'_>,
) -> Result<Result<ExternalUser, PluginError>>
```

Decode a `ExternalUserReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_event_poll_result

```rust
pub fn write_event_poll_result(b: Builder<'_>, v: &EventPollResult) -> Result<()>
```

Encode a [`EventPollResult`](#eventpollresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_event_poll_result

```rust
pub fn read_event_poll_result(r: Reader<'_>) -> Result<EventPollResult>
```

Decode a [`EventPollResult`](#eventpollresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_event_poll_reply

```rust
pub fn write_event_poll_reply(
    b: Builder<'_>,
    v: &Result<EventPollResult, PluginError>,
) -> Result<()>
```

Encode a `EventPollReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_event_poll_reply

```rust
pub fn read_event_poll_reply(
    r: Reader<'_>,
) -> Result<Result<EventPollResult, PluginError>>
```

Decode a `EventPollReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_listening_progress

```rust
pub fn write_listening_progress(b: Builder<'_>, v: &ListeningProgress) -> Result<()>
```

Encode a [`ListeningProgress`](#listeningprogress) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_listening_progress

```rust
pub fn read_listening_progress(r: Reader<'_>) -> Result<ListeningProgress>
```

Decode a [`ListeningProgress`](#listeningprogress) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_sync_listening_result

```rust
pub fn write_sync_listening_result(b: Builder<'_>, v: &SyncListeningResult) -> Result<()>
```

Encode a [`SyncListeningResult`](#synclisteningresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_sync_listening_result

```rust
pub fn read_sync_listening_result(r: Reader<'_>) -> Result<SyncListeningResult>
```

Decode a [`SyncListeningResult`](#synclisteningresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_sync_listening_reply

```rust
pub fn write_sync_listening_reply(
    b: Builder<'_>,
    v: &Result<SyncListeningResult, PluginError>,
) -> Result<()>
```

Encode a `SyncListeningReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_sync_listening_reply

```rust
pub fn read_sync_listening_reply(
    r: Reader<'_>,
) -> Result<Result<SyncListeningResult, PluginError>>
```

Decode a `SyncListeningReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_plugin_event

```rust
pub fn write_plugin_event(b: Builder<'_>, v: &PluginEvent) -> Result<()>
```

Encode a [`PluginEvent`](#pluginevent) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_plugin_event

```rust
pub fn read_plugin_event(r: Reader<'_>) -> Result<PluginEvent>
```

Decode a [`PluginEvent`](#pluginevent) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_publish_ok

```rust
pub fn write_publish_ok(b: Builder<'_>, v: &PublishOk) -> Result<()>
```

Encode a [`PublishOk`](#publishok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_publish_ok

```rust
pub fn read_publish_ok(r: Reader<'_>) -> Result<PublishOk>
```

Decode a [`PublishOk`](#publishok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_publish_reply

```rust
pub fn write_publish_reply(
    b: Builder<'_>,
    v: &Result<PublishOk, PluginError>,
) -> Result<()>
```

Encode a `PublishReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_publish_reply

```rust
pub fn read_publish_reply(r: Reader<'_>) -> Result<Result<PublishOk, PluginError>>
```

Decode a `PublishReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_invocation

```rust
pub fn write_invocation(b: Builder<'_>, v: &Invocation) -> Result<()>
```

Encode a [`Invocation`](#invocation) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_invocation

```rust
pub fn read_invocation(r: Reader<'_>) -> Result<Invocation>
```

Decode a [`Invocation`](#invocation) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_message

```rust
pub fn write_queue_message(b: Builder<'_>, v: &QueueMessage) -> Result<()>
```

Encode a [`QueueMessage`](#queuemessage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_message

```rust
pub fn read_queue_message(r: Reader<'_>) -> Result<QueueMessage>
```

Decode a [`QueueMessage`](#queuemessage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_batch

```rust
pub fn write_queue_batch(b: Builder<'_>, v: &QueueBatch) -> Result<()>
```

Encode a [`QueueBatch`](#queuebatch) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_batch

```rust
pub fn read_queue_batch(r: Reader<'_>) -> Result<QueueBatch>
```

Decode a [`QueueBatch`](#queuebatch) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_retry

```rust
pub fn write_queue_retry(b: Builder<'_>, v: &QueueRetry) -> Result<()>
```

Encode a [`QueueRetry`](#queueretry) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_retry

```rust
pub fn read_queue_retry(r: Reader<'_>) -> Result<QueueRetry>
```

Decode a [`QueueRetry`](#queueretry) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_result

```rust
pub fn write_queue_result(b: Builder<'_>, v: &QueueResult) -> Result<()>
```

Encode a `QueueResult` union.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_queue_result

```rust
pub fn read_queue_result(r: Reader<'_>) -> Result<QueueResult>
```

Decode a `QueueResult` union.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_queue_batch_ok

```rust
pub fn write_queue_batch_ok(b: Builder<'_>, v: &QueueBatchOk) -> Result<()>
```

Encode a [`QueueBatchOk`](#queuebatchok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_batch_ok

```rust
pub fn read_queue_batch_ok(r: Reader<'_>) -> Result<QueueBatchOk>
```

Decode a [`QueueBatchOk`](#queuebatchok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_batch_reply

```rust
pub fn write_queue_batch_reply(
    b: Builder<'_>,
    v: &Result<QueueBatchOk, PluginError>,
) -> Result<()>
```

Encode a `QueueBatchReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_queue_batch_reply

```rust
pub fn read_queue_batch_reply(r: Reader<'_>) -> Result<Result<QueueBatchOk, PluginError>>
```

Decode a `QueueBatchReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_queue_send_message

```rust
pub fn write_queue_send_message(b: Builder<'_>, v: &QueueSendMessage) -> Result<()>
```

Encode a [`QueueSendMessage`](#queuesendmessage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_send_message

```rust
pub fn read_queue_send_message(r: Reader<'_>) -> Result<QueueSendMessage>
```

Decode a [`QueueSendMessage`](#queuesendmessage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_send_ok

```rust
pub fn write_queue_send_ok(b: Builder<'_>, v: &QueueSendOk) -> Result<()>
```

Encode a [`QueueSendOk`](#queuesendok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_queue_send_ok

```rust
pub fn read_queue_send_ok(r: Reader<'_>) -> Result<QueueSendOk>
```

Decode a [`QueueSendOk`](#queuesendok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_queue_send_reply

```rust
pub fn write_queue_send_reply(
    b: Builder<'_>,
    v: &Result<QueueSendOk, PluginError>,
) -> Result<()>
```

Encode a `QueueSendReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_queue_send_reply

```rust
pub fn read_queue_send_reply(r: Reader<'_>) -> Result<Result<QueueSendOk, PluginError>>
```

Decode a `QueueSendReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_kv_put_options

```rust
pub fn write_kv_put_options(b: Builder<'_>, v: &KvPutOptions) -> Result<()>
```

Encode a [`KvPutOptions`](#kvputoptions) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_put_options

```rust
pub fn read_kv_put_options(r: Reader<'_>) -> Result<KvPutOptions>
```

Decode a [`KvPutOptions`](#kvputoptions) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_get_ok

```rust
pub fn write_kv_get_ok(b: Builder<'_>, v: &KvGetOk) -> Result<()>
```

Encode a [`KvGetOk`](#kvgetok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_get_ok

```rust
pub fn read_kv_get_ok(r: Reader<'_>) -> Result<KvGetOk>
```

Decode a [`KvGetOk`](#kvgetok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_get_reply

```rust
pub fn write_kv_get_reply(b: Builder<'_>, v: &Result<KvGetOk, PluginError>) -> Result<()>
```

Encode a `KvGetReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_kv_get_reply

```rust
pub fn read_kv_get_reply(r: Reader<'_>) -> Result<Result<KvGetOk, PluginError>>
```

Decode a `KvGetReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_kv_put_ok

```rust
pub fn write_kv_put_ok(b: Builder<'_>, v: &KvPutOk) -> Result<()>
```

Encode a [`KvPutOk`](#kvputok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_put_ok

```rust
pub fn read_kv_put_ok(r: Reader<'_>) -> Result<KvPutOk>
```

Decode a [`KvPutOk`](#kvputok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_delete_ok

```rust
pub fn write_kv_delete_ok(b: Builder<'_>, v: &KvDeleteOk) -> Result<()>
```

Encode a [`KvDeleteOk`](#kvdeleteok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_delete_ok

```rust
pub fn read_kv_delete_ok(r: Reader<'_>) -> Result<KvDeleteOk>
```

Decode a [`KvDeleteOk`](#kvdeleteok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_put_reply

```rust
pub fn write_kv_put_reply(b: Builder<'_>, v: &Result<KvPutOk, PluginError>) -> Result<()>
```

Encode a `KvPutReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_kv_put_reply

```rust
pub fn read_kv_put_reply(r: Reader<'_>) -> Result<Result<KvPutOk, PluginError>>
```

Decode a `KvPutReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_kv_delete_reply

```rust
pub fn write_kv_delete_reply(
    b: Builder<'_>,
    v: &Result<KvDeleteOk, PluginError>,
) -> Result<()>
```

Encode a `KvDeleteReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_kv_delete_reply

```rust
pub fn read_kv_delete_reply(r: Reader<'_>) -> Result<Result<KvDeleteOk, PluginError>>
```

Decode a `KvDeleteReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_kv_list_options

```rust
pub fn write_kv_list_options(b: Builder<'_>, v: &KvListOptions) -> Result<()>
```

Encode a [`KvListOptions`](#kvlistoptions) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_list_options

```rust
pub fn read_kv_list_options(r: Reader<'_>) -> Result<KvListOptions>
```

Decode a [`KvListOptions`](#kvlistoptions) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_list_key

```rust
pub fn write_kv_list_key(b: Builder<'_>, v: &KvListKey) -> Result<()>
```

Encode a [`KvListKey`](#kvlistkey) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_list_key

```rust
pub fn read_kv_list_key(r: Reader<'_>) -> Result<KvListKey>
```

Decode a [`KvListKey`](#kvlistkey) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_list_ok

```rust
pub fn write_kv_list_ok(b: Builder<'_>, v: &KvListOk) -> Result<()>
```

Encode a [`KvListOk`](#kvlistok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_kv_list_ok

```rust
pub fn read_kv_list_ok(r: Reader<'_>) -> Result<KvListOk>
```

Decode a [`KvListOk`](#kvlistok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_kv_list_reply

```rust
pub fn write_kv_list_reply(
    b: Builder<'_>,
    v: &Result<KvListOk, PluginError>,
) -> Result<()>
```

Encode a `KvListReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_kv_list_reply

```rust
pub fn read_kv_list_reply(r: Reader<'_>) -> Result<Result<KvListOk, PluginError>>
```

Decode a `KvListReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### read_any_message

```rust
pub fn read_any_message(selector: u8, bytes: &[u8]) -> Result<()>
```

Decodes `bytes` as an unpacked single-segment Cap'n Proto message whose
root is the struct `selector` names, through that struct's `read_*`
codec, and drops the result.

Harness entry point for the `abi_decode` fuzz target and its corpus
replay: every generated decoder runs with a traversal budget derived
from the input size and a fixed nesting limit, so a small buffer cannot
expand into a large graph.

#### Errors

Returns the Cap'n Proto error the framing or the codec reports, or a
failure for a selector at or past [`READ_ANY_COUNT`](#read_any_count).

### roundtrip_any_message

```rust
pub fn roundtrip_any_message(selector: u8, bytes: &[u8]) -> Result<(), RoundTripFailure>
```

Decodes `bytes` as the struct `selector` names (as [`read_any_message`](#read_any_message)
does), writes the value back through that struct's `write_*` codec,
decodes the result and requires it to equal the first value.

Harness entry point for the `abi_roundtrip` fuzz target and its corpus
replay. The first decode runs under the same traversal and nesting
budget as [`read_any_message`](#read_any_message); the re-decode runs on bytes this crate
produced from an already bounded value, so it uses the default budget.
Equality is the DTO's `PartialEq`; because `NaN != NaN`, two values
that `PartialEq` separates still count as equal when their `Debug`
renderings agree (a field-by-field comparison that prints every NaN
the same way and keeps `-0.0` apart from `0.0`) and the re-decoded
value re-encodes to the same bytes (which also covers fields whose
`Debug` redacts).

#### Errors

[`RoundTripFailure::Rejected`](#roundtripfailure) when the input does not decode or the
selector is at or past [`READ_ANY_COUNT`](#read_any_count); [`RoundTripFailure::Codec`](#roundtripfailure)
when the decoded value does not survive `write_*` then `read_*` intact.

## Constants

### READ_ANY_COUNT

```rust
pub const READ_ANY_COUNT: u8 = 82;
```

Number of root structs [`read_any_message`](#read_any_message) can decode; selectors are
`0..READ_ANY_COUNT` in schema order.
