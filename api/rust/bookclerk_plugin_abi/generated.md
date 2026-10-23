**bookclerk_plugin_abi > generated**

# Module: generated

## Contents

**Structs**

- [`AccountCredential`](#accountcredential) - Host-sealed credentials for one account, delivered on `scan`.
- [`AuthenticateUserParams`](#authenticateuserparams) - Params of `Oidc.authenticateUser`.
- [`Brand`](#brand) - Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
- [`CatalogDetail`](#catalogdetail) - Success payload of `ContentSource.catalogDetail`.
- [`CatalogDetailParams`](#catalogdetailparams) - Params of `ContentSource.catalogDetail`.
- [`CatalogHit`](#cataloghit) - One catalog / candidate hit returned by `searchCatalog`,
- [`CatalogHits`](#cataloghits) - Success payload of the catalog list methods.
- [`ChapterMarker`](#chaptermarker) - One chapter marker of a fetched title.
- [`CliArg`](#cliarg) - One named argument value passed to `PluginCli.invoke`.
- [`CliArgSpec`](#cliargspec) - One CLI argument or flag under a `CliCommandSpec`.
- [`CliCommandSpec`](#clicommandspec) - One plugin CLI command under `CliSchema`.
- [`CliInvokeParams`](#cliinvokeparams) - Params of `PluginCli.invoke`.
- [`CliInvokeResult`](#cliinvokeresult) - Result of `PluginCli.invoke`.
- [`CliSchema`](#clischema) - Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).
- [`ConfigOption`](#configoption) - One discoverable config option group advertised for sources.
- [`ConfigOptionValue`](#configoptionvalue) - One selectable value under a `ConfigOption`.
- [`DatabaseAdapterConfig`](#databaseadapterconfig) - Author-facing database adapter configuration carried in
- [`DiagnoseResult`](#diagnoseresult) - Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.
- [`EventConsumerSpec`](#eventconsumerspec) - One declared event consumer: a `\[\[events.consumers\]\]` row the default
- [`EventPollResult`](#eventpollresult) - Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
- [`ExpandCandidatesParams`](#expandcandidatesparams) - Params of `ContentSource.expandCandidates`. Seed fields identify a known
- [`ExternalUser`](#externaluser) - One external user observed by an integration. The host may mint claim
- [`FetchOptions`](#fetchoptions) - Fetch-relevant acquire knobs the host forwards so external load matches
- [`FetchTitleParams`](#fetchtitleparams) - Params of `ContentSource.fetchTitle`. The plugin writes media under
- [`Invocation`](#invocation) - Identity of one `PluginWorker.open` invocation. The host issues ids; guests
- [`KvDeleteOk`](#kvdeleteok) - Success payload of `KvNamespace.delete`.
- [`KvGetOk`](#kvgetok) - Success payload of `KvNamespace.get`.
- [`KvListKey`](#kvlistkey) - One key on a `KvNamespace.list` page.
- [`KvListOk`](#kvlistok) - Success payload of `KvNamespace.list`.
- [`KvListOptions`](#kvlistoptions) - Page request for `KvNamespace.list`.
- [`KvPutOk`](#kvputok) - Success payload of `KvNamespace.put`.
- [`KvPutOptions`](#kvputoptions) - Expiration and metadata for `KvNamespace.put`.
- [`ListDealsParams`](#listdealsparams) - Params of `ContentSource.listDeals`.
- [`ListeningProgress`](#listeningprogress) - One listening-progress row. The host upserts into `listening_progress`
- [`LoginCompleteParams`](#logincompleteparams) - Params of `ContentSource.loginComplete`.
- [`LoginParams`](#loginparams) - Params of `ContentSource.login` and `ContentSource.loginStart`. Password
- [`LoginResult`](#loginresult) - Result of `ContentSource.login` / `loginComplete`: account metadata plus
- [`LoginStartResult`](#loginstartresult) - Result of `ContentSource.loginStart` (interactive OAuth). The operator
- [`PlainFetch`](#plainfetch) - Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
- [`PlainPart`](#plainpart) - One plain audio part written under the cache directory.
- [`PluginCapabilities`](#plugincapabilities) - Typed capability declaration returned by `describe()`. The host compares
- [`PluginEvent`](#pluginevent) - Guest-published domain event (`EventPublisher.publish`). The host stamps
- [`PublishOk`](#publishok) - Success payload of `EventPublisher.publish`.
- [`PurchaseHint`](#purchasehint) - Purchase hint for SPA / CLI purchase deep-links.
- [`PurchaseHintParams`](#purchasehintparams) - Params of `ContentSource.purchaseHint`. At least one identity field
- [`PurchaseHintResult`](#purchasehintresult) - Success payload of `ContentSource.purchaseHint`.
- [`QueueBatch`](#queuebatch) - One `QueueConsumer.queue` delivery: messages from one queue in send order.
- [`QueueBatchOk`](#queuebatchok) - Success payload of `QueueConsumer.queue`: exactly one entry per
- [`QueueMessage`](#queuemessage) - One message on a Workers-style queue, delivered by `QueueConsumer.queue`.
- [`QueueRetry`](#queueretry) - Redeliver a queue message later.
- [`QueueSendMessage`](#queuesendmessage) - One message to send through a `\[\[queues.producers\]\]` binding.
- [`QueueSendOk`](#queuesendok) - Success payload of `QueueProducer.send` / `sendBatch`.
- [`ScanBook`](#scanbook) - One library title returned by `ContentSource.scan`. The host upserts these
- [`ScanLibraryParams`](#scanlibraryparams) - Params of `RemoteLibrary.scanLibrary` (remote library sync).
- [`ScanParams`](#scanparams) - Params of `ContentSource.scan`. The host injects sealed credentials so the
- [`ScanSummary`](#scansummary) - Summary result of `ContentSource.scan`.
- [`SearchCatalogParams`](#searchcatalogparams) - Params of `ContentSource.searchCatalog`.
- [`SourceAccount`](#sourceaccount) - Source account metadata returned from login and stored by the host.
- [`SourceAccounts`](#sourceaccounts) - Success payload of `ContentSource.listAccounts`.
- [`SyncListeningResult`](#synclisteningresult) - Success payload of `RemoteLibrary.syncListening`.

**Enums**

- [`Abridgement`](#abridgement) - Whether an edition is abridged.
- [`CatalogField`](#catalogfield) - Catalog search facet restricting which field the query matches.
- [`CatalogSort`](#catalogsort) - Catalog search ordering.
- [`CliArgKind`](#cliargkind) - Value kind for a `CliArgSpec`.
- [`Entrypoint`](#entrypoint) - Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
- [`PortalAuthMode`](#portalauthmode) - Portal Accounts connect mode for storefronts.
- [`QueueResult`](#queueresult) - Per-message outcome inside a `QueueConsumer.queue` batch reply.
- [`RoundTripFailure`](#roundtripfailure) - Why [`roundtrip_any_message`](generated.md#bookclerk_plugin_abigeneratedroundtrip_any_message) could not show `read(write(v)) == v`.

**Functions**

- [`read_account_credential`](#read_account_credential) - Decode a [`AccountCredential`](generated.md#bookclerk_plugin_abigeneratedaccountcredential) from a Cap'n Proto reader.
- [`read_any_message`](#read_any_message) - Decodes `bytes` as an unpacked single-segment Cap'n Proto message whose
- [`read_authenticate_user_params`](#read_authenticate_user_params) - Decode a [`AuthenticateUserParams`](generated.md#bookclerk_plugin_abigeneratedauthenticateuserparams) from a Cap'n Proto reader.
- [`read_brand`](#read_brand) - Decode a [`Brand`](generated.md#bookclerk_plugin_abigeneratedbrand) from a Cap'n Proto reader.
- [`read_catalog_detail`](#read_catalog_detail) - Decode a [`CatalogDetail`](generated.md#bookclerk_plugin_abigeneratedcatalogdetail) from a Cap'n Proto reader.
- [`read_catalog_detail_params`](#read_catalog_detail_params) - Decode a [`CatalogDetailParams`](generated.md#bookclerk_plugin_abigeneratedcatalogdetailparams) from a Cap'n Proto reader.
- [`read_catalog_detail_reply`](#read_catalog_detail_reply) - Decode a `CatalogDetailReply` union into a typed result.
- [`read_catalog_hit`](#read_catalog_hit) - Decode a [`CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit) from a Cap'n Proto reader.
- [`read_catalog_hits`](#read_catalog_hits) - Decode a [`CatalogHits`](generated.md#bookclerk_plugin_abigeneratedcataloghits) from a Cap'n Proto reader.
- [`read_catalog_hits_reply`](#read_catalog_hits_reply) - Decode a `CatalogHitsReply` union into a typed result.
- [`read_chapter_marker`](#read_chapter_marker) - Decode a [`ChapterMarker`](generated.md#bookclerk_plugin_abigeneratedchaptermarker) from a Cap'n Proto reader.
- [`read_cli_arg`](#read_cli_arg) - Decode a [`CliArg`](generated.md#bookclerk_plugin_abigeneratedcliarg) from a Cap'n Proto reader.
- [`read_cli_arg_spec`](#read_cli_arg_spec) - Decode a [`CliArgSpec`](generated.md#bookclerk_plugin_abigeneratedcliargspec) from a Cap'n Proto reader.
- [`read_cli_command_spec`](#read_cli_command_spec) - Decode a [`CliCommandSpec`](generated.md#bookclerk_plugin_abigeneratedclicommandspec) from a Cap'n Proto reader.
- [`read_cli_invoke_params`](#read_cli_invoke_params) - Decode a [`CliInvokeParams`](generated.md#bookclerk_plugin_abigeneratedcliinvokeparams) from a Cap'n Proto reader.
- [`read_cli_invoke_reply`](#read_cli_invoke_reply) - Decode a `CliInvokeReply` union into a typed result.
- [`read_cli_invoke_result`](#read_cli_invoke_result) - Decode a [`CliInvokeResult`](generated.md#bookclerk_plugin_abigeneratedcliinvokeresult) from a Cap'n Proto reader.
- [`read_cli_schema`](#read_cli_schema) - Decode a [`CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema) from a Cap'n Proto reader.
- [`read_cli_schema_reply`](#read_cli_schema_reply) - Decode a `CliSchemaReply` union into a typed result.
- [`read_config_option`](#read_config_option) - Decode a [`ConfigOption`](generated.md#bookclerk_plugin_abigeneratedconfigoption) from a Cap'n Proto reader.
- [`read_config_option_value`](#read_config_option_value) - Decode a [`ConfigOptionValue`](generated.md#bookclerk_plugin_abigeneratedconfigoptionvalue) from a Cap'n Proto reader.
- [`read_database_adapter_config`](#read_database_adapter_config) - Decode a [`DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) from a Cap'n Proto reader.
- [`read_diagnose_reply`](#read_diagnose_reply) - Decode a `DiagnoseReply` union into a typed result.
- [`read_diagnose_result`](#read_diagnose_result) - Decode a [`DiagnoseResult`](generated.md#bookclerk_plugin_abigenerateddiagnoseresult) from a Cap'n Proto reader.
- [`read_event_consumer_spec`](#read_event_consumer_spec) - Decode a [`EventConsumerSpec`](generated.md#bookclerk_plugin_abigeneratedeventconsumerspec) from a Cap'n Proto reader.
- [`read_event_poll_reply`](#read_event_poll_reply) - Decode a `EventPollReply` union into a typed result.
- [`read_event_poll_result`](#read_event_poll_result) - Decode a [`EventPollResult`](generated.md#bookclerk_plugin_abigeneratedeventpollresult) from a Cap'n Proto reader.
- [`read_expand_candidates_params`](#read_expand_candidates_params) - Decode a [`ExpandCandidatesParams`](generated.md#bookclerk_plugin_abigeneratedexpandcandidatesparams) from a Cap'n Proto reader.
- [`read_external_user`](#read_external_user) - Decode a [`ExternalUser`](generated.md#bookclerk_plugin_abigeneratedexternaluser) from a Cap'n Proto reader.
- [`read_external_user_reply`](#read_external_user_reply) - Decode a `ExternalUserReply` union into a typed result.
- [`read_fetch_options`](#read_fetch_options) - Decode a [`FetchOptions`](generated.md#bookclerk_plugin_abigeneratedfetchoptions) from a Cap'n Proto reader.
- [`read_fetch_title_params`](#read_fetch_title_params) - Decode a [`FetchTitleParams`](generated.md#bookclerk_plugin_abigeneratedfetchtitleparams) from a Cap'n Proto reader.
- [`read_fetch_title_reply`](#read_fetch_title_reply) - Decode a `FetchTitleReply` union into a typed result.
- [`read_invocation`](#read_invocation) - Decode a [`Invocation`](generated.md#bookclerk_plugin_abigeneratedinvocation) from a Cap'n Proto reader.
- [`read_kv_delete_ok`](#read_kv_delete_ok) - Decode a [`KvDeleteOk`](generated.md#bookclerk_plugin_abigeneratedkvdeleteok) from a Cap'n Proto reader.
- [`read_kv_delete_reply`](#read_kv_delete_reply) - Decode a `KvDeleteReply` union into a typed result.
- [`read_kv_get_ok`](#read_kv_get_ok) - Decode a [`KvGetOk`](generated.md#bookclerk_plugin_abigeneratedkvgetok) from a Cap'n Proto reader.
- [`read_kv_get_reply`](#read_kv_get_reply) - Decode a `KvGetReply` union into a typed result.
- [`read_kv_list_key`](#read_kv_list_key) - Decode a [`KvListKey`](generated.md#bookclerk_plugin_abigeneratedkvlistkey) from a Cap'n Proto reader.
- [`read_kv_list_ok`](#read_kv_list_ok) - Decode a [`KvListOk`](generated.md#bookclerk_plugin_abigeneratedkvlistok) from a Cap'n Proto reader.
- [`read_kv_list_options`](#read_kv_list_options) - Decode a [`KvListOptions`](generated.md#bookclerk_plugin_abigeneratedkvlistoptions) from a Cap'n Proto reader.
- [`read_kv_list_reply`](#read_kv_list_reply) - Decode a `KvListReply` union into a typed result.
- [`read_kv_put_ok`](#read_kv_put_ok) - Decode a [`KvPutOk`](generated.md#bookclerk_plugin_abigeneratedkvputok) from a Cap'n Proto reader.
- [`read_kv_put_options`](#read_kv_put_options) - Decode a [`KvPutOptions`](generated.md#bookclerk_plugin_abigeneratedkvputoptions) from a Cap'n Proto reader.
- [`read_kv_put_reply`](#read_kv_put_reply) - Decode a `KvPutReply` union into a typed result.
- [`read_list_deals_params`](#read_list_deals_params) - Decode a [`ListDealsParams`](generated.md#bookclerk_plugin_abigeneratedlistdealsparams) from a Cap'n Proto reader.
- [`read_listening_progress`](#read_listening_progress) - Decode a [`ListeningProgress`](generated.md#bookclerk_plugin_abigeneratedlisteningprogress) from a Cap'n Proto reader.
- [`read_login_complete_params`](#read_login_complete_params) - Decode a [`LoginCompleteParams`](generated.md#bookclerk_plugin_abigeneratedlogincompleteparams) from a Cap'n Proto reader.
- [`read_login_params`](#read_login_params) - Decode a [`LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams) from a Cap'n Proto reader.
- [`read_login_reply`](#read_login_reply) - Decode a `LoginReply` union into a typed result.
- [`read_login_result`](#read_login_result) - Decode a [`LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult) from a Cap'n Proto reader.
- [`read_login_start_reply`](#read_login_start_reply) - Decode a `LoginStartReply` union into a typed result.
- [`read_login_start_result`](#read_login_start_result) - Decode a [`LoginStartResult`](generated.md#bookclerk_plugin_abigeneratedloginstartresult) from a Cap'n Proto reader.
- [`read_plain_fetch`](#read_plain_fetch) - Decode a [`PlainFetch`](generated.md#bookclerk_plugin_abigeneratedplainfetch) from a Cap'n Proto reader.
- [`read_plain_part`](#read_plain_part) - Decode a [`PlainPart`](generated.md#bookclerk_plugin_abigeneratedplainpart) from a Cap'n Proto reader.
- [`read_plugin_capabilities`](#read_plugin_capabilities) - Decode a [`PluginCapabilities`](generated.md#bookclerk_plugin_abigeneratedplugincapabilities) from a Cap'n Proto reader.
- [`read_plugin_event`](#read_plugin_event) - Decode a [`PluginEvent`](generated.md#bookclerk_plugin_abigeneratedpluginevent) from a Cap'n Proto reader.
- [`read_publish_ok`](#read_publish_ok) - Decode a [`PublishOk`](generated.md#bookclerk_plugin_abigeneratedpublishok) from a Cap'n Proto reader.
- [`read_publish_reply`](#read_publish_reply) - Decode a `PublishReply` union into a typed result.
- [`read_purchase_hint`](#read_purchase_hint) - Decode a [`PurchaseHint`](generated.md#bookclerk_plugin_abigeneratedpurchasehint) from a Cap'n Proto reader.
- [`read_purchase_hint_params`](#read_purchase_hint_params) - Decode a [`PurchaseHintParams`](generated.md#bookclerk_plugin_abigeneratedpurchasehintparams) from a Cap'n Proto reader.
- [`read_purchase_hint_reply`](#read_purchase_hint_reply) - Decode a `PurchaseHintReply` union into a typed result.
- [`read_purchase_hint_result`](#read_purchase_hint_result) - Decode a [`PurchaseHintResult`](generated.md#bookclerk_plugin_abigeneratedpurchasehintresult) from a Cap'n Proto reader.
- [`read_queue_batch`](#read_queue_batch) - Decode a [`QueueBatch`](generated.md#bookclerk_plugin_abigeneratedqueuebatch) from a Cap'n Proto reader.
- [`read_queue_batch_ok`](#read_queue_batch_ok) - Decode a [`QueueBatchOk`](generated.md#bookclerk_plugin_abigeneratedqueuebatchok) from a Cap'n Proto reader.
- [`read_queue_batch_reply`](#read_queue_batch_reply) - Decode a `QueueBatchReply` union into a typed result.
- [`read_queue_message`](#read_queue_message) - Decode a [`QueueMessage`](generated.md#bookclerk_plugin_abigeneratedqueuemessage) from a Cap'n Proto reader.
- [`read_queue_result`](#read_queue_result) - Decode a `QueueResult` union.
- [`read_queue_retry`](#read_queue_retry) - Decode a [`QueueRetry`](generated.md#bookclerk_plugin_abigeneratedqueueretry) from a Cap'n Proto reader.
- [`read_queue_send_message`](#read_queue_send_message) - Decode a [`QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) from a Cap'n Proto reader.
- [`read_queue_send_ok`](#read_queue_send_ok) - Decode a [`QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok) from a Cap'n Proto reader.
- [`read_queue_send_reply`](#read_queue_send_reply) - Decode a `QueueSendReply` union into a typed result.
- [`read_scan_book`](#read_scan_book) - Decode a [`ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) from a Cap'n Proto reader.
- [`read_scan_library_params`](#read_scan_library_params) - Decode a [`ScanLibraryParams`](generated.md#bookclerk_plugin_abigeneratedscanlibraryparams) from a Cap'n Proto reader.
- [`read_scan_params`](#read_scan_params) - Decode a [`ScanParams`](generated.md#bookclerk_plugin_abigeneratedscanparams) from a Cap'n Proto reader.
- [`read_scan_reply`](#read_scan_reply) - Decode a `ScanReply` union into a typed result.
- [`read_scan_summary`](#read_scan_summary) - Decode a [`ScanSummary`](generated.md#bookclerk_plugin_abigeneratedscansummary) from a Cap'n Proto reader.
- [`read_search_catalog_params`](#read_search_catalog_params) - Decode a [`SearchCatalogParams`](generated.md#bookclerk_plugin_abigeneratedsearchcatalogparams) from a Cap'n Proto reader.
- [`read_source_account`](#read_source_account) - Decode a [`SourceAccount`](generated.md#bookclerk_plugin_abigeneratedsourceaccount) from a Cap'n Proto reader.
- [`read_source_accounts`](#read_source_accounts) - Decode a [`SourceAccounts`](generated.md#bookclerk_plugin_abigeneratedsourceaccounts) from a Cap'n Proto reader.
- [`read_source_accounts_reply`](#read_source_accounts_reply) - Decode a `SourceAccountsReply` union into a typed result.
- [`read_sync_listening_reply`](#read_sync_listening_reply) - Decode a `SyncListeningReply` union into a typed result.
- [`read_sync_listening_result`](#read_sync_listening_result) - Decode a [`SyncListeningResult`](generated.md#bookclerk_plugin_abigeneratedsynclisteningresult) from a Cap'n Proto reader.
- [`roundtrip_any_message`](#roundtrip_any_message) - Decodes `bytes` as the struct `selector` names (as [`read_any_message`](generated.md#bookclerk_plugin_abigeneratedread_any_message)
- [`write_account_credential`](#write_account_credential) - Encode a [`AccountCredential`](generated.md#bookclerk_plugin_abigeneratedaccountcredential) onto a Cap'n Proto builder.
- [`write_authenticate_user_params`](#write_authenticate_user_params) - Encode a [`AuthenticateUserParams`](generated.md#bookclerk_plugin_abigeneratedauthenticateuserparams) onto a Cap'n Proto builder.
- [`write_brand`](#write_brand) - Encode a [`Brand`](generated.md#bookclerk_plugin_abigeneratedbrand) onto a Cap'n Proto builder.
- [`write_catalog_detail`](#write_catalog_detail) - Encode a [`CatalogDetail`](generated.md#bookclerk_plugin_abigeneratedcatalogdetail) onto a Cap'n Proto builder.
- [`write_catalog_detail_params`](#write_catalog_detail_params) - Encode a [`CatalogDetailParams`](generated.md#bookclerk_plugin_abigeneratedcatalogdetailparams) onto a Cap'n Proto builder.
- [`write_catalog_detail_reply`](#write_catalog_detail_reply) - Encode a `CatalogDetailReply` union from a typed result.
- [`write_catalog_hit`](#write_catalog_hit) - Encode a [`CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit) onto a Cap'n Proto builder.
- [`write_catalog_hits`](#write_catalog_hits) - Encode a [`CatalogHits`](generated.md#bookclerk_plugin_abigeneratedcataloghits) onto a Cap'n Proto builder.
- [`write_catalog_hits_reply`](#write_catalog_hits_reply) - Encode a `CatalogHitsReply` union from a typed result.
- [`write_chapter_marker`](#write_chapter_marker) - Encode a [`ChapterMarker`](generated.md#bookclerk_plugin_abigeneratedchaptermarker) onto a Cap'n Proto builder.
- [`write_cli_arg`](#write_cli_arg) - Encode a [`CliArg`](generated.md#bookclerk_plugin_abigeneratedcliarg) onto a Cap'n Proto builder.
- [`write_cli_arg_spec`](#write_cli_arg_spec) - Encode a [`CliArgSpec`](generated.md#bookclerk_plugin_abigeneratedcliargspec) onto a Cap'n Proto builder.
- [`write_cli_command_spec`](#write_cli_command_spec) - Encode a [`CliCommandSpec`](generated.md#bookclerk_plugin_abigeneratedclicommandspec) onto a Cap'n Proto builder.
- [`write_cli_invoke_params`](#write_cli_invoke_params) - Encode a [`CliInvokeParams`](generated.md#bookclerk_plugin_abigeneratedcliinvokeparams) onto a Cap'n Proto builder.
- [`write_cli_invoke_reply`](#write_cli_invoke_reply) - Encode a `CliInvokeReply` union from a typed result.
- [`write_cli_invoke_result`](#write_cli_invoke_result) - Encode a [`CliInvokeResult`](generated.md#bookclerk_plugin_abigeneratedcliinvokeresult) onto a Cap'n Proto builder.
- [`write_cli_schema`](#write_cli_schema) - Encode a [`CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema) onto a Cap'n Proto builder.
- [`write_cli_schema_reply`](#write_cli_schema_reply) - Encode a `CliSchemaReply` union from a typed result.
- [`write_config_option`](#write_config_option) - Encode a [`ConfigOption`](generated.md#bookclerk_plugin_abigeneratedconfigoption) onto a Cap'n Proto builder.
- [`write_config_option_value`](#write_config_option_value) - Encode a [`ConfigOptionValue`](generated.md#bookclerk_plugin_abigeneratedconfigoptionvalue) onto a Cap'n Proto builder.
- [`write_database_adapter_config`](#write_database_adapter_config) - Encode a [`DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) onto a Cap'n Proto builder.
- [`write_diagnose_reply`](#write_diagnose_reply) - Encode a `DiagnoseReply` union from a typed result.
- [`write_diagnose_result`](#write_diagnose_result) - Encode a [`DiagnoseResult`](generated.md#bookclerk_plugin_abigenerateddiagnoseresult) onto a Cap'n Proto builder.
- [`write_event_consumer_spec`](#write_event_consumer_spec) - Encode a [`EventConsumerSpec`](generated.md#bookclerk_plugin_abigeneratedeventconsumerspec) onto a Cap'n Proto builder.
- [`write_event_poll_reply`](#write_event_poll_reply) - Encode a `EventPollReply` union from a typed result.
- [`write_event_poll_result`](#write_event_poll_result) - Encode a [`EventPollResult`](generated.md#bookclerk_plugin_abigeneratedeventpollresult) onto a Cap'n Proto builder.
- [`write_expand_candidates_params`](#write_expand_candidates_params) - Encode a [`ExpandCandidatesParams`](generated.md#bookclerk_plugin_abigeneratedexpandcandidatesparams) onto a Cap'n Proto builder.
- [`write_external_user`](#write_external_user) - Encode a [`ExternalUser`](generated.md#bookclerk_plugin_abigeneratedexternaluser) onto a Cap'n Proto builder.
- [`write_external_user_reply`](#write_external_user_reply) - Encode a `ExternalUserReply` union from a typed result.
- [`write_fetch_options`](#write_fetch_options) - Encode a [`FetchOptions`](generated.md#bookclerk_plugin_abigeneratedfetchoptions) onto a Cap'n Proto builder.
- [`write_fetch_title_params`](#write_fetch_title_params) - Encode a [`FetchTitleParams`](generated.md#bookclerk_plugin_abigeneratedfetchtitleparams) onto a Cap'n Proto builder.
- [`write_fetch_title_reply`](#write_fetch_title_reply) - Encode a `FetchTitleReply` union from a typed result.
- [`write_invocation`](#write_invocation) - Encode a [`Invocation`](generated.md#bookclerk_plugin_abigeneratedinvocation) onto a Cap'n Proto builder.
- [`write_kv_delete_ok`](#write_kv_delete_ok) - Encode a [`KvDeleteOk`](generated.md#bookclerk_plugin_abigeneratedkvdeleteok) onto a Cap'n Proto builder.
- [`write_kv_delete_reply`](#write_kv_delete_reply) - Encode a `KvDeleteReply` union from a typed result.
- [`write_kv_get_ok`](#write_kv_get_ok) - Encode a [`KvGetOk`](generated.md#bookclerk_plugin_abigeneratedkvgetok) onto a Cap'n Proto builder.
- [`write_kv_get_reply`](#write_kv_get_reply) - Encode a `KvGetReply` union from a typed result.
- [`write_kv_list_key`](#write_kv_list_key) - Encode a [`KvListKey`](generated.md#bookclerk_plugin_abigeneratedkvlistkey) onto a Cap'n Proto builder.
- [`write_kv_list_ok`](#write_kv_list_ok) - Encode a [`KvListOk`](generated.md#bookclerk_plugin_abigeneratedkvlistok) onto a Cap'n Proto builder.
- [`write_kv_list_options`](#write_kv_list_options) - Encode a [`KvListOptions`](generated.md#bookclerk_plugin_abigeneratedkvlistoptions) onto a Cap'n Proto builder.
- [`write_kv_list_reply`](#write_kv_list_reply) - Encode a `KvListReply` union from a typed result.
- [`write_kv_put_ok`](#write_kv_put_ok) - Encode a [`KvPutOk`](generated.md#bookclerk_plugin_abigeneratedkvputok) onto a Cap'n Proto builder.
- [`write_kv_put_options`](#write_kv_put_options) - Encode a [`KvPutOptions`](generated.md#bookclerk_plugin_abigeneratedkvputoptions) onto a Cap'n Proto builder.
- [`write_kv_put_reply`](#write_kv_put_reply) - Encode a `KvPutReply` union from a typed result.
- [`write_list_deals_params`](#write_list_deals_params) - Encode a [`ListDealsParams`](generated.md#bookclerk_plugin_abigeneratedlistdealsparams) onto a Cap'n Proto builder.
- [`write_listening_progress`](#write_listening_progress) - Encode a [`ListeningProgress`](generated.md#bookclerk_plugin_abigeneratedlisteningprogress) onto a Cap'n Proto builder.
- [`write_login_complete_params`](#write_login_complete_params) - Encode a [`LoginCompleteParams`](generated.md#bookclerk_plugin_abigeneratedlogincompleteparams) onto a Cap'n Proto builder.
- [`write_login_params`](#write_login_params) - Encode a [`LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams) onto a Cap'n Proto builder.
- [`write_login_reply`](#write_login_reply) - Encode a `LoginReply` union from a typed result.
- [`write_login_result`](#write_login_result) - Encode a [`LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult) onto a Cap'n Proto builder.
- [`write_login_start_reply`](#write_login_start_reply) - Encode a `LoginStartReply` union from a typed result.
- [`write_login_start_result`](#write_login_start_result) - Encode a [`LoginStartResult`](generated.md#bookclerk_plugin_abigeneratedloginstartresult) onto a Cap'n Proto builder.
- [`write_plain_fetch`](#write_plain_fetch) - Encode a [`PlainFetch`](generated.md#bookclerk_plugin_abigeneratedplainfetch) onto a Cap'n Proto builder.
- [`write_plain_part`](#write_plain_part) - Encode a [`PlainPart`](generated.md#bookclerk_plugin_abigeneratedplainpart) onto a Cap'n Proto builder.
- [`write_plugin_capabilities`](#write_plugin_capabilities) - Encode a [`PluginCapabilities`](generated.md#bookclerk_plugin_abigeneratedplugincapabilities) onto a Cap'n Proto builder.
- [`write_plugin_event`](#write_plugin_event) - Encode a [`PluginEvent`](generated.md#bookclerk_plugin_abigeneratedpluginevent) onto a Cap'n Proto builder.
- [`write_publish_ok`](#write_publish_ok) - Encode a [`PublishOk`](generated.md#bookclerk_plugin_abigeneratedpublishok) onto a Cap'n Proto builder.
- [`write_publish_reply`](#write_publish_reply) - Encode a `PublishReply` union from a typed result.
- [`write_purchase_hint`](#write_purchase_hint) - Encode a [`PurchaseHint`](generated.md#bookclerk_plugin_abigeneratedpurchasehint) onto a Cap'n Proto builder.
- [`write_purchase_hint_params`](#write_purchase_hint_params) - Encode a [`PurchaseHintParams`](generated.md#bookclerk_plugin_abigeneratedpurchasehintparams) onto a Cap'n Proto builder.
- [`write_purchase_hint_reply`](#write_purchase_hint_reply) - Encode a `PurchaseHintReply` union from a typed result.
- [`write_purchase_hint_result`](#write_purchase_hint_result) - Encode a [`PurchaseHintResult`](generated.md#bookclerk_plugin_abigeneratedpurchasehintresult) onto a Cap'n Proto builder.
- [`write_queue_batch`](#write_queue_batch) - Encode a [`QueueBatch`](generated.md#bookclerk_plugin_abigeneratedqueuebatch) onto a Cap'n Proto builder.
- [`write_queue_batch_ok`](#write_queue_batch_ok) - Encode a [`QueueBatchOk`](generated.md#bookclerk_plugin_abigeneratedqueuebatchok) onto a Cap'n Proto builder.
- [`write_queue_batch_reply`](#write_queue_batch_reply) - Encode a `QueueBatchReply` union from a typed result.
- [`write_queue_message`](#write_queue_message) - Encode a [`QueueMessage`](generated.md#bookclerk_plugin_abigeneratedqueuemessage) onto a Cap'n Proto builder.
- [`write_queue_result`](#write_queue_result) - Encode a `QueueResult` union.
- [`write_queue_retry`](#write_queue_retry) - Encode a [`QueueRetry`](generated.md#bookclerk_plugin_abigeneratedqueueretry) onto a Cap'n Proto builder.
- [`write_queue_send_message`](#write_queue_send_message) - Encode a [`QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) onto a Cap'n Proto builder.
- [`write_queue_send_ok`](#write_queue_send_ok) - Encode a [`QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok) onto a Cap'n Proto builder.
- [`write_queue_send_reply`](#write_queue_send_reply) - Encode a `QueueSendReply` union from a typed result.
- [`write_scan_book`](#write_scan_book) - Encode a [`ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) onto a Cap'n Proto builder.
- [`write_scan_library_params`](#write_scan_library_params) - Encode a [`ScanLibraryParams`](generated.md#bookclerk_plugin_abigeneratedscanlibraryparams) onto a Cap'n Proto builder.
- [`write_scan_params`](#write_scan_params) - Encode a [`ScanParams`](generated.md#bookclerk_plugin_abigeneratedscanparams) onto a Cap'n Proto builder.
- [`write_scan_reply`](#write_scan_reply) - Encode a `ScanReply` union from a typed result.
- [`write_scan_summary`](#write_scan_summary) - Encode a [`ScanSummary`](generated.md#bookclerk_plugin_abigeneratedscansummary) onto a Cap'n Proto builder.
- [`write_search_catalog_params`](#write_search_catalog_params) - Encode a [`SearchCatalogParams`](generated.md#bookclerk_plugin_abigeneratedsearchcatalogparams) onto a Cap'n Proto builder.
- [`write_source_account`](#write_source_account) - Encode a [`SourceAccount`](generated.md#bookclerk_plugin_abigeneratedsourceaccount) onto a Cap'n Proto builder.
- [`write_source_accounts`](#write_source_accounts) - Encode a [`SourceAccounts`](generated.md#bookclerk_plugin_abigeneratedsourceaccounts) onto a Cap'n Proto builder.
- [`write_source_accounts_reply`](#write_source_accounts_reply) - Encode a `SourceAccountsReply` union from a typed result.
- [`write_sync_listening_reply`](#write_sync_listening_reply) - Encode a `SyncListeningReply` union from a typed result.
- [`write_sync_listening_result`](#write_sync_listening_result) - Encode a [`SyncListeningResult`](generated.md#bookclerk_plugin_abigeneratedsynclisteningresult) onto a Cap'n Proto builder.

**Constants**

- [`READ_ANY_COUNT`](#read_any_count) - Number of root structs [`read_any_message`](generated.md#bookclerk_plugin_abigeneratedread_any_message) can decode; selectors are

---

## bookclerk_plugin_abi::generated::Abridgement

*Enum*

Whether an edition is abridged.

**Variants:**
- `Unknown` - The storefront did not say.
- `Unabridged` - Unabridged edition.
- `Abridged` - Abridged edition.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Eq, Copy

**Trait Implementations:**

- **From**
  - `fn from(value: plugin_capnp::Abridgement) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Abridgement) -> bool`
- **Default**
  - `fn default() -> Abridgement`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Abridgement`



## bookclerk_plugin_abi::generated::AccountCredential

*Struct*

Host-sealed credentials for one account, delivered on `scan`.

**Fields:**
- `account_id: String` - Account id the blob belongs to.
- `credentials: Vec<u8>` - Opaque credential bytes exactly as the guest returned them at login.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &AccountCredential) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AccountCredential`
- **Default**
  - `fn default() -> AccountCredential`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::AuthenticateUserParams

*Struct*

Params of `Oidc.authenticateUser`.

**Fields:**
- `username: String` - Integration username / login id.
- `password: String` - Integration password; never logged by the host.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &AuthenticateUserParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> AuthenticateUserParams`
- **Default**
  - `fn default() -> AuthenticateUserParams`



## bookclerk_plugin_abi::generated::Brand

*Struct*

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

**Fields:**
- `id: String` - Brand id (often matches the plugin id); empty means "no brand".
- `name: String` - Display name shown next to the brand swatch.
- `bg: String` - Background CSS color (hex or named).
- `fg: String` - Foreground CSS color for text on `bg`.
- `accent: String` - Accent CSS color for highlights / CTAs.
- `icon_url: Option<String>` - Icon URL or data URI for the portal.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Brand`
- **Default**
  - `fn default() -> Brand`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &Brand) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::CatalogDetail

*Struct*

Success payload of `ContentSource.catalogDetail`.

**Fields:**
- `found: bool` - False when the product is unknown to the storefront (`hit` is empty).
- `hit: CatalogHit` - Full catalog record when `found`.

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogDetail`
- **Default**
  - `fn default() -> CatalogDetail`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogDetail) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::CatalogDetailParams

*Struct*

Params of `ContentSource.catalogDetail`.

**Fields:**
- `product_id: String` - Store product id (Libro ISBN or ISBN-slug).
- `isbn: Option<String>` - ISBN when it differs from `productId`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CatalogDetailParams`
- **Default**
  - `fn default() -> CatalogDetailParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogDetailParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::CatalogField

*Enum*

Catalog search facet restricting which field the query matches.

**Variants:**
- `Any` - Match any field.
- `Author` - Match author names.
- `Narrator` - Match narrator names.
- `Series` - Match series names.
- `Genre` - Match genre / category labels.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> CatalogField`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogField`
- **From**
  - `fn from(value: plugin_capnp::CatalogField) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogField) -> bool`



## bookclerk_plugin_abi::generated::CatalogHit

*Struct*

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

**Fields:**
- `product_id: String` - Storefront product / SKU id.
- `title: String` - Primary title.
- `authors: Option<String>` - Authors string when known.
- `narrators: Option<String>` - Narrators string when known.
- `series: Option<String>` - Series name when applicable.
- `series_index: Option<String>` - Series index / sequence label.
- `asin: Option<String>` - Amazon ASIN when known.
- `isbn: Option<String>` - ISBN when known.
- `url: Option<String>` - Storefront product page URL.
- `cover_url: Option<String>` - Cover image URL.
- `origin: String` - Hit origin label (plugin id or storefront name).
- `subtitle: Option<String>` - Subtitle when distinct from `title`.
- `description: Option<String>` - Long description / blurb when fetched.
- `publisher: Option<String>` - Publisher name when known.
- `length_minutes: Option<i64>` - Runtime in whole minutes.
- `published_at: Option<String>` - Publication date string as provided by the storefront.
- `categories: Option<String>` - Category / genre labels as a single string when known.
- `language: Option<String>` - Content language code when known.
- `price_cents: Option<i64>` - Current price in minor units (cents).
- `currency: Option<String>` - ISO currency code for `priceCents`.
- `price_label: Option<String>` - Pre-formatted price for display.
- `rating_overall: Option<f64>` - Aggregate rating when known.
- `rating_count: Option<i64>` - Number of ratings when known.
- `abridgement: Abridgement` - Whether the edition is abridged when the storefront says so.

**Trait Implementations:**

- **Default**
  - `fn default() -> CatalogHit`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogHit) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogHit`



## bookclerk_plugin_abi::generated::CatalogHits

*Struct*

Success payload of the catalog list methods.

**Fields:**
- `hits: Vec<CatalogHit>` - Hits in storefront order.

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogHits) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogHits`
- **Default**
  - `fn default() -> CatalogHits`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::generated::CatalogSort

*Enum*

Catalog search ordering.

**Variants:**
- `Relevance` - Storefront relevance ranking (default).
- `Popularity` - Most popular first.
- `Rating` - Highest rated first.
- `Title` - Alphabetical by title.
- `Author` - Alphabetical by author.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CatalogSort`
- **From**
  - `fn from(value: plugin_capnp::CatalogSort) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CatalogSort) -> bool`
- **Default**
  - `fn default() -> CatalogSort`



## bookclerk_plugin_abi::generated::ChapterMarker

*Struct*

One chapter marker of a fetched title.

**Fields:**
- `title: String` - Chapter title.
- `start_ms: u64` - Chapter start offset in milliseconds from the beginning of the title.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ChapterMarker) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ChapterMarker`
- **Default**
  - `fn default() -> ChapterMarker`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::CliArg

*Struct*

One named argument value passed to `PluginCli.invoke`.

**Fields:**
- `name: String` - Arg name matching a `CliArgSpec.name`.
- `value: String` - String form of the value (the guest parses per `CliArgSpec.kind`).

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliArg`
- **Default**
  - `fn default() -> CliArg`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliArg) -> bool`



## bookclerk_plugin_abi::generated::CliArgKind

*Enum*

Value kind for a `CliArgSpec`.

**Variants:**
- `String` - Free-form string argument (default).
- `Bool` - Boolean flag ("true" / "false").
- `Int` - Integer argument.
- `Path` - Filesystem path argument.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliArgKind`
- **From**
  - `fn from(value: plugin_capnp::CliArgKind) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliArgKind) -> bool`
- **Default**
  - `fn default() -> CliArgKind`



## bookclerk_plugin_abi::generated::CliArgSpec

*Struct*

One CLI argument or flag under a `CliCommandSpec`.

**Fields:**
- `name: String` - Internal arg name used as `CliArg.name` on invoke.
- `long: Option<String>` - Long flag without leading dashes (e.g. "message" -> `--message`).
- `short: Option<String>` - Short flag character (e.g. "m" -> `-m`).
- `kind: CliArgKind` - Parsed value kind.
- `required: bool` - When true, the host rejects invoke if the arg is missing.
- `default: Option<String>` - Default string form when the operator omits the arg.
- `about: Option<String>` - Help text for this arg.
- `positional: bool` - When true, the arg is positional rather than a flagged option.

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> CliArgSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliArgSpec) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliArgSpec`



## bookclerk_plugin_abi::generated::CliCommandSpec

*Struct*

One plugin CLI command under `CliSchema`.

**Fields:**
- `name: String` - Command verb after the plugin id (for example "ping").
- `about: Option<String>` - Short help text for `--help`.
- `args: Vec<CliArgSpec>` - Argument / flag specs for this command.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliCommandSpec`
- **Default**
  - `fn default() -> CliCommandSpec`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliCommandSpec) -> bool`



## bookclerk_plugin_abi::generated::CliInvokeParams

*Struct*

Params of `PluginCli.invoke`.

**Fields:**
- `command: String` - Command name matching a `CliCommandSpec.name`.
- `args: Vec<CliArg>` - Named argument values.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliInvokeParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliInvokeParams`
- **Default**
  - `fn default() -> CliInvokeParams`



## bookclerk_plugin_abi::generated::CliInvokeResult

*Struct*

Result of `PluginCli.invoke`.

**Fields:**
- `exit_code: i32` - Process-style exit code (0 = success).
- `stdout: String` - Captured standard output text.
- `stderr: String` - Captured standard error text.
- `payload: crate::ExtensibleConfig` - Structured payload for machine consumers; empty `mediaType` when absent.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliInvokeResult) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> CliInvokeResult`
- **Default**
  - `fn default() -> CliInvokeResult`



## bookclerk_plugin_abi::generated::CliSchema

*Struct*

Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).

**Fields:**
- `commands: Vec<CliCommandSpec>` - Commands exposed as `bookclerk plugins <id> <command> ...`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CliSchema`
- **Default**
  - `fn default() -> CliSchema`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CliSchema) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::ConfigOption

*Struct*

One discoverable config option group advertised for sources.

**Fields:**
- `key: String` - Config key under the plugin's `config.toml` table.
- `label: String` - Operator-facing label for the option group.
- `values: Vec<ConfigOptionValue>` - Allowed selectable values for this key.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ConfigOption`
- **Default**
  - `fn default() -> ConfigOption`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ConfigOption) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::ConfigOptionValue

*Struct*

One selectable value under a `ConfigOption`.

**Fields:**
- `id: String` - Value written to config when selected.
- `label: String` - Operator-facing label for this value.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ConfigOptionValue) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ConfigOptionValue`
- **Default**
  - `fn default() -> ConfigOptionValue`



## bookclerk_plugin_abi::generated::DatabaseAdapterConfig

*Struct*

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `\[database.<id>\]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

**Fields:**
- `plugin_data_dir: String` - Scoped writable directory for this plugin (`.../plugins/<id>/data`).
- `settings: crate::ExtensibleConfig` - Granted plugin settings (operator `\[database.<id>\]` table) as
- `binding: Option<String>` - Named plugin database binding this open serves; empty for the primary
- `instance_id: Option<String>` - Host-issued opaque instance id for this (owner plugin, binding) pair.
- `open_existing: bool` - When true, open an existing binding unit and do not provision a missing

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DatabaseAdapterConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DatabaseAdapterConfig`
- **Default**
  - `fn default() -> DatabaseAdapterConfig`



## bookclerk_plugin_abi::generated::DiagnoseResult

*Struct*

Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.

**Fields:**
- `lines: Vec<String>` - Human-readable probe lines.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> DiagnoseResult`
- **Default**
  - `fn default() -> DiagnoseResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &DiagnoseResult) -> bool`



## bookclerk_plugin_abi::generated::Entrypoint

*Enum*

Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
analogue). Each value is a capability the host calls over RPC; triggers on
the default entrypoint (`event`, `job`) are declared separately.

**Variants:**
- `Storefront` - Storefront: login, scan, fetch, catalog search (`ContentSource`).
- `Storage` - Object storage destination (`Destination`).
- `DatabaseAdapter` - Library database adapter (`Database`).
- `RemoteLibrary` - Remote-library lifecycle: start/stop, scanLibrary, syncListening,
- `Cli` - Guest CLI (`PluginCli.describe` / `PluginCli.invoke`).
- `Oidc` - OIDC bridge: relying-party client templates and `authenticateUser`.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> Entrypoint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Entrypoint) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **From**
  - `fn from(value: plugin_capnp::Entrypoint) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Entrypoint`



## bookclerk_plugin_abi::generated::EventConsumerSpec

*Struct*

One declared event consumer: a `\[\[events.consumers\]\]` row the default
entrypoint's `event(batch)` handler accepts.

**Fields:**
- `event_type: String` - Versioned event type (snake_case, e.g. `book_acquired`).
- `schema_versions: Vec<u32>` - Schema versions the guest can consume; never empty.
- `supports_suspend: bool` - Whether `EventResult.suspended` is supported for this type.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventConsumerSpec) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> EventConsumerSpec`
- **Default**
  - `fn default() -> EventConsumerSpec`



## bookclerk_plugin_abi::generated::EventPollResult

*Struct*

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

**Fields:**
- `users: Vec<ExternalUser>` - Newly observed external users since the last poll.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &EventPollResult) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> EventPollResult`
- **Default**
  - `fn default() -> EventPollResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::ExpandCandidatesParams

*Struct*

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

**Fields:**
- `source: String` - Source plugin id hint when expanding across storefronts.
- `product_id: String` - Seed storefront product id.
- `title: String` - Seed title text.
- `authors: Option<String>` - Seed authors string.
- `narrators: Option<String>` - Seed narrators string.
- `series: Option<String>` - Seed series name.
- `series_asin: Option<String>` - Seed series ASIN when known.
- `asin: Option<String>` - Seed Amazon ASIN.
- `isbn: Option<String>` - Seed ISBN.
- `region: String` - Storefront region / marketplace code.
- `limit: u32` - Maximum candidates to return; the host always sends an explicit value.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ExpandCandidatesParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ExpandCandidatesParams`
- **Default**
  - `fn default() -> ExpandCandidatesParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::ExternalUser

*Struct*

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

**Fields:**
- `provider: String` - Integration provider id (often the plugin id).
- `external_user_id: String` - Provider-scoped user id.
- `display_name: Option<String>` - Display name for UI.
- `access_token: Option<String>` - Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ExternalUser) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ExternalUser`
- **Default**
  - `fn default() -> ExternalUser`



## bookclerk_plugin_abi::generated::FetchOptions

*Struct*

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

**Fields:**
- `widevine: bool` - Prefer Widevine/CENC download when the store offers it.
- `xhe_aac: bool` - Prefer xHE-AAC on the Widevine path when offered.
- `widevine_cdm_path: Option<String>` - Local Widevine `.wvd` path granted to the guest.
- `widevine_cdm_provider: Option<String>` - Remote L3 CDM provider URL; empty means the classic default, `off`
- `download_cover: bool` - When true, download a cover image alongside audio.
- `download_pdf: bool` - When true, download a companion PDF when the store exposes one.
- `cover_size: String` - Cover image size request (`500`, `1215`, or `native`).
- `chapter_layout: String` - Preferred chapter API layout when fetching (`tree` or `flat`).
- `strip_audible_brand_audio: bool` - When true, trim Audible brand intro/outro from the remux window.
- `download_clips_bookmarks: bool` - When true, download clips/bookmarks sidecars when offered.
- `retain_aax_file: bool` - When true, keep the encrypted download in storage.
- `download_speed_limit_kbps: u32` - Fetch speed cap in KB/s (`0` = unlimited).
- `save_metadata_json: bool` - When true, persist raw catalog API JSON as `metadata.json`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> FetchOptions`
- **Default**
  - `fn default() -> FetchOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &FetchOptions) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::FetchTitleParams

*Struct*

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

**Fields:**
- `plugin_data_dir: String` - Scoped plugin data directory.
- `account_id: String` - Account whose credentials apply.
- `title_id: String` - Library / storefront title id to download.
- `cache_dir: String` - Absolute path the guest should write media into (jail-granted TMPDIR).
- `credentials: Option<Vec<u8>>` - Host-loaded credential blob for this account; empty when unavailable.
- `source_config: crate::ExtensibleConfig` - Granted `\[sources.<id>\]` table as `application/json`.
- `fetch: FetchOptions` - Fetch-relevant acquire options.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &FetchTitleParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> FetchTitleParams`
- **Default**
  - `fn default() -> FetchTitleParams`



## bookclerk_plugin_abi::generated::Invocation

*Struct*

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

**Fields:**
- `id: String` - Unique host-issued invocation id.
- `account_id: String` - Account scope; empty for operator / host-wide invocations.
- `deadline_unix_ms: u64` - UTC Unix milliseconds; zero when the invocation has no deadline. The host
- `correlation_id: String` - Trace correlation id; empty when none.
- `causation_id: String` - Id of the event or command that caused this invocation; empty when none.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> Invocation`
- **Default**
  - `fn default() -> Invocation`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &Invocation) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::KvDeleteOk

*Struct*

Success payload of `KvNamespace.delete`.

**Fields:**
- `existed: bool` - True when a live value existed before the delete.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvDeleteOk`
- **Default**
  - `fn default() -> KvDeleteOk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvDeleteOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::KvGetOk

*Struct*

Success payload of `KvNamespace.get`.

**Fields:**
- `found: bool` - True when the key exists and has not expired.
- `value: Vec<u8>` - Stored value; empty when not found.
- `metadata: String` - Stored metadata JSON text; empty when none.
- `expiration_unix_ms: u64` - Absolute expiry (UTC Unix milliseconds); zero means none.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> KvGetOk`
- **Default**
  - `fn default() -> KvGetOk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvGetOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::KvListKey

*Struct*

One key on a `KvNamespace.list` page.

**Fields:**
- `key: String` - Key name.
- `metadata: String` - Stored metadata JSON text; empty when none.
- `expiration_unix_ms: u64` - Absolute expiry (UTC Unix milliseconds); zero means none.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvListKey) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvListKey`
- **Default**
  - `fn default() -> KvListKey`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::generated::KvListOk

*Struct*

Success payload of `KvNamespace.list`.

**Fields:**
- `keys: Vec<KvListKey>` - Keys in lexical order.
- `list_complete: bool` - True when no further page exists.
- `cursor: String` - Cursor for the next page; empty when `listComplete`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvListOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvListOk`
- **Default**
  - `fn default() -> KvListOk`



## bookclerk_plugin_abi::generated::KvListOptions

*Struct*

Page request for `KvNamespace.list`.

**Fields:**
- `prefix: String` - Only keys starting with this prefix; empty means all.
- `cursor: String` - Opaque cursor from a previous page; empty means the first page.
- `limit: u32` - Most keys to return; zero means the host default, at most `maxListPage`.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvListOptions`
- **Default**
  - `fn default() -> KvListOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvListOptions) -> bool`



## bookclerk_plugin_abi::generated::KvPutOk

*Struct*

Success payload of `KvNamespace.put`.

**Fields:**
- `expiration_unix_ms: u64` - Effective absolute expiry (UTC Unix milliseconds); zero means none.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &KvPutOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvPutOk`
- **Default**
  - `fn default() -> KvPutOk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::KvPutOptions

*Struct*

Expiration and metadata for `KvNamespace.put`.

**Fields:**
- `expiration_unix_ms: u64` - Absolute expiry (UTC Unix milliseconds); zero means none.
- `expiration_ttl_seconds: u64` - Relative expiry in seconds from the write; zero means none. Ignored when
- `metadata: String` - Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &KvPutOptions) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> KvPutOptions`
- **Default**
  - `fn default() -> KvPutOptions`



## bookclerk_plugin_abi::generated::ListDealsParams

*Struct*

Params of `ContentSource.listDeals`.

**Fields:**
- `limit: Option<u32>` - Maximum number of deals to return; guest default when 0.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListDealsParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListDealsParams`
- **Default**
  - `fn default() -> ListDealsParams`



## bookclerk_plugin_abi::generated::ListeningProgress

*Struct*

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

**Fields:**
- `external_user_id: String` - Provider-scoped user id.
- `external_item_id: String` - Provider-scoped item / library id.
- `identity_id: Option<i64>` - Bookclerk identity row id when already linked.
- `title: Option<String>` - Title text when known.
- `authors: Option<String>` - Authors string when known.
- `asin: Option<String>` - Amazon ASIN when known.
- `isbn: Option<String>` - ISBN when known.
- `progress: Option<f64>` - Fractional progress in `0.0..=1.0` when the provider reports it.
- `current_time_seconds: Option<f64>` - Current playback position in seconds.
- `duration_seconds: Option<f64>` - Total duration in seconds when known.
- `is_finished: bool` - When true, the provider marks the item finished.
- `last_listened_at_unix_ms: Option<u64>` - Last listen timestamp as unix milliseconds (UTC).

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ListeningProgress`
- **Default**
  - `fn default() -> ListeningProgress`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &ListeningProgress) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::LoginCompleteParams

*Struct*

Params of `ContentSource.loginComplete`.

**Fields:**
- `session_id: String` - Session id previously returned by `loginStart`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LoginCompleteParams`
- **Default**
  - `fn default() -> LoginCompleteParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &LoginCompleteParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::LoginParams

*Struct*

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

**Fields:**
- `plugin_data_dir: String` - Scoped writable directory for this plugin only (`.../plugins/<id>/data`).
- `marketplace: String` - Marketplace / locale for the storefront; empty means the guest default.
- `label: Option<String>` - Operator label stored on the account row.
- `email: Option<String>` - Account email / username for password logins; empty for pure OAuth.
- `password: Option<String>` - Account password for password logins; never logged; empty for OAuth.
- `force: bool` - When true, overwrite an existing sealed credential for this account.
- `callback_bind: Option<String>` - Bind address for OAuth callback servers (`host:port`). Ignored when
- `callback_ipc: Option<String>` - Host-owned callback IPC endpoint the guest must connect to. When set
- `callback_public_base: Option<String>` - Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`.
- `external: bool` - When true, use external / paste-redirect OAuth instead of a local
- `response_url: Option<String>` - Pre-supplied OAuth redirect URL (paste flow).
- `show_qr: bool` - Prefer QR output when the guest supports it.
- `timeout_secs: Option<u64>` - Seconds to wait for OAuth callback capture; guest default when 0.
- `extra: crate::ExtensibleConfig` - Store-specific knobs as `application/json`; guests may ignore unknowns.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &LoginParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LoginParams`
- **Default**
  - `fn default() -> LoginParams`



## bookclerk_plugin_abi::generated::LoginResult

*Struct*

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

**Fields:**
- `account: SourceAccount` - Account row fields for the host to upsert.
- `credentials: Option<Vec<u8>>` - Opaque credential blob the host seals; empty when login only refreshed

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LoginResult`
- **Default**
  - `fn default() -> LoginResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &LoginResult) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::LoginStartResult

*Struct*

Result of `ContentSource.loginStart` (interactive OAuth). The operator
opens `url`; `loginComplete` later uses `sessionId`.

**Fields:**
- `session_id: String` - Opaque session id for `loginComplete`.
- `url: String` - Browser URL the operator should open to complete OAuth.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &LoginStartResult) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> LoginStartResult`
- **Default**
  - `fn default() -> LoginStartResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::PlainFetch

*Struct*

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

**Fields:**
- `parts: Vec<PlainPart>` - Ordered audio part files written under the cache directory.
- `m4b_path: Option<String>` - Single M4B path when the guest assembled one.
- `cover_path: Option<String>` - Cover image path under the cache directory.
- `chapters: Vec<ChapterMarker>` - Chapter markers; empty when unknown.
- `pdf_url: Option<String>` - Companion PDF download URL when the store exposes one.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PlainFetch`
- **Default**
  - `fn default() -> PlainFetch`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlainFetch) -> bool`



## bookclerk_plugin_abi::generated::PlainPart

*Struct*

One plain audio part written under the cache directory.

**Fields:**
- `path: String` - Absolute path to the part file under `cacheDir`.
- `title: Option<String>` - Part title (disc / chapter label).
- `duration_ms: Option<u64>` - Duration of this part in milliseconds when known.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PlainPart`
- **Default**
  - `fn default() -> PlainPart`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PlainPart) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::PluginCapabilities

*Struct*

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

**Fields:**
- `entrypoints: Vec<Entrypoint>` - Named entrypoints the guest exports.
- `consumes: Vec<EventConsumerSpec>` - Event types the default entrypoint consumes (`event(batch)` trigger).
- `produces: Vec<String>` - Event types the guest may publish through its `EVENTS` binding.
- `jobs: Vec<String>` - Command types the default entrypoint runs (`job(controller)` trigger).
- `databases: Vec<String>` - Plugin-owned database binding names (`\[\[databases\]\]`).
- `bindings: Vec<String>` - Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`,
- `queue_consumers: Vec<String>` - Queues the default entrypoint consumes (`queue(batch)` trigger).
- `queue_producers: Vec<String>` - Queues the guest may send to through its `\[\[queues.producers\]\]` bindings.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginCapabilities`
- **Default**
  - `fn default() -> PluginCapabilities`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginCapabilities) -> bool`



## bookclerk_plugin_abi::generated::PluginEvent

*Struct*

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

**Fields:**
- `event_type: String` - Snake_case event type; must be listed in `\[\[events.producers\]\]`.
- `schema_version: u32` - Schema version of `payload`, owned by the event type.
- `deduplication_key: String` - Producer idempotency key, unique per (account, source, eventType) in the
- `payload: Vec<u8>` - Encoded event payload; at most `maxEventPayloadBytes`.
- `occurred_at_unix_ms: u64` - When the producer observed the fact; zero means "now" on the host clock.
- `correlation_id: String` - Trace correlation id; empty inherits `Invocation.correlationId`.
- `causation_id: String` - Id of the event or command that caused this one; empty inherits

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PluginEvent) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PluginEvent`
- **Default**
  - `fn default() -> PluginEvent`



## bookclerk_plugin_abi::generated::PortalAuthMode

*Enum*

Portal Accounts connect mode for storefronts.

**Variants:**
- `Unspecified` - Guest did not declare a mode; the host assumes password login.
- `Password` - Email / password login through the Accounts UI.
- `Oauth` - Browser OAuth through `loginStart` / `loginComplete`.

**Methods:**

- `fn wire_name(self: Self) -> &'static str` - Wire name of this enumerant (matches the TypeScript / Python SDK literal).
- `fn from_wire_name(name: &str) -> Option<Self>` - Parse a wire name; `None` for unknown strings.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PortalAuthMode`
- **From**
  - `fn from(value: plugin_capnp::PortalAuthMode) -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &PortalAuthMode) -> bool`
- **Default**
  - `fn default() -> PortalAuthMode`
- **Hash**
  - `fn hash<__H>(self: &Self, state: & mut __H)`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::PublishOk

*Struct*

Success payload of `EventPublisher.publish`.

**Fields:**
- `event_id: String` - Outbox event id (new or the earlier row when `duplicate`).
- `duplicate: bool` - True when `deduplicationKey` matched an existing outbox row.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PublishOk`
- **Default**
  - `fn default() -> PublishOk`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PublishOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::PurchaseHint

*Struct*

Purchase hint for SPA / CLI purchase deep-links.

**Fields:**
- `product_id: String` - Storefront product id.
- `title: Option<String>` - Title when resolved.
- `url: Option<String>` - Purchase or product-page URL.
- `price_cents: Option<i64>` - Current price in minor units.
- `currency: Option<String>` - ISO currency code for price fields.
- `price_label: Option<String>` - Pre-formatted current price.
- `list_price_cents: Option<i64>` - List / MSRP price in minor units.
- `list_price_label: Option<String>` - Pre-formatted list price.
- `member_price_cents: Option<i64>` - Member / Plus price in minor units.
- `member_price_label: Option<String>` - Pre-formatted member price.

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHint`
- **Default**
  - `fn default() -> PurchaseHint`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PurchaseHint) -> bool`



## bookclerk_plugin_abi::generated::PurchaseHintParams

*Struct*

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

**Fields:**
- `product_id: Option<String>` - Storefront product id when known.
- `title: Option<String>` - Title text for fuzzy lookup.
- `authors: Option<String>` - Authors string for fuzzy lookup.
- `asin: Option<String>` - Amazon ASIN when known.
- `isbn: Option<String>` - ISBN when known.
- `region: String` - Storefront region / marketplace code.
- `with_price: bool` - When true, guests should include live price fields when available.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> PurchaseHintParams`
- **Default**
  - `fn default() -> PurchaseHintParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PurchaseHintParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::PurchaseHintResult

*Struct*

Success payload of `ContentSource.purchaseHint`.

**Fields:**
- `found: bool` - False when the guest could not resolve the title (`hint` is empty).
- `hint: PurchaseHint` - Purchase hint when `found`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &PurchaseHintResult) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> PurchaseHintResult`
- **Default**
  - `fn default() -> PurchaseHintResult`



## bookclerk_plugin_abi::generated::QueueBatch

*Struct*

One `QueueConsumer.queue` delivery: messages from one queue in send order.

**Fields:**
- `queue: String` - Queue the messages were sent to.
- `messages: Vec<QueueMessage>` - Messages in delivery order; the reply carries one `QueueResult` per entry.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> QueueBatch`
- **Default**
  - `fn default() -> QueueBatch`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueBatch) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::QueueBatchOk

*Struct*

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

**Fields:**
- `results: Vec<QueueResult>` - Per-message outcomes in delivery order.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueBatchOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueBatchOk`
- **Default**
  - `fn default() -> QueueBatchOk`



## bookclerk_plugin_abi::generated::QueueMessage

*Struct*

One message on a Workers-style queue, delivered by `QueueConsumer.queue`.

**Fields:**
- `id: String` - Outbox row id; stable across redeliveries.
- `timestamp_unix_ms: u64` - When the producer sent the message (UTC Unix milliseconds).
- `body: Vec<u8>` - Encoded body; at most `maxEventPayloadBytes`.
- `content_type: String` - Body encoding: `json`, `text`, or `bytes`.
- `attempts: u32` - Delivery counter, starting at 1.
- `source: String` - Producer plugin id; empty when the host sent it.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueMessage) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueMessage`
- **Default**
  - `fn default() -> QueueMessage`



## bookclerk_plugin_abi::generated::QueueResult

*Enum*

Per-message outcome inside a `QueueConsumer.queue` batch reply.

**Variants:**
- `Ack` - Handled; the host marks the message delivered.
- `Retry(QueueRetry)` - Redeliver later.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueResult) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueResult`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::QueueRetry

*Struct*

Redeliver a queue message later.

**Fields:**
- `delay_seconds: u32` - Seconds before redelivery; zero applies the consumer's `retry_delay`.
- `reason: String` - Short human reason.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueRetry) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueRetry`
- **Default**
  - `fn default() -> QueueRetry`



## bookclerk_plugin_abi::generated::QueueSendMessage

*Struct*

One message to send through a `\[\[queues.producers\]\]` binding.

**Fields:**
- `body: Vec<u8>` - Encoded body; at most `maxEventPayloadBytes`.
- `content_type: String` - Body encoding: `json` (default when empty), `text`, or `bytes`.
- `delay_seconds: u32` - Seconds before the message may be delivered; zero means now.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> QueueSendMessage`
- **Default**
  - `fn default() -> QueueSendMessage`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueSendMessage) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::QueueSendOk

*Struct*

Success payload of `QueueProducer.send` / `sendBatch`.

**Fields:**
- `message_ids: Vec<String>` - Outbox row ids in send order.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &QueueSendOk) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> QueueSendOk`
- **Default**
  - `fn default() -> QueueSendOk`



## bookclerk_plugin_abi::generated::READ_ANY_COUNT

*Constant*: `u8`

Number of root structs [`read_any_message`](generated.md#bookclerk_plugin_abigeneratedread_any_message) can decode; selectors are
`0..READ_ANY_COUNT` in schema order.



## bookclerk_plugin_abi::generated::RoundTripFailure

*Enum*

Why [`roundtrip_any_message`](generated.md#bookclerk_plugin_abigeneratedroundtrip_any_message) could not show `read(write(v)) == v`.

**Variants:**
- `Rejected(capnp::Error)` - The input bytes did not decode; expected for arbitrary input and
- `Codec(String)` - A value that did decode failed to re-encode, to decode again, or

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **From**
  - `fn from(err: capnp::Error) -> Self`



## bookclerk_plugin_abi::generated::ScanBook

*Struct*

One library title returned by `ContentSource.scan`. The host upserts these
rows and forces `source` to the plugin id.

**Fields:**
- `account_id: String` - Account that owns this library entry.
- `product_id: String` - Storefront product / SKU id.
- `title: String` - Primary title string.
- `marketplace: Option<String>` - Marketplace / region when known.
- `asin: Option<String>` - Amazon ASIN when the storefront exposes one.
- `isbn: Option<String>` - ISBN when the storefront exposes one.
- `authors: Option<String>` - Comma- or guest-formatted author list.
- `narrators: Option<String>` - Comma- or guest-formatted narrator list.
- `series: Option<String>` - Series name when applicable.
- `series_index: Option<String>` - Series index / sequence label.
- `content_kind: Option<String>` - Content classification (e.g. `book` vs `episode`).
- `publisher: Option<String>` - Publisher name when known.
- `length_minutes: Option<i64>` - Runtime in whole minutes when known.
- `subtitle: Option<String>` - Subtitle when distinct from `title`.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanBook) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ScanBook`
- **Default**
  - `fn default() -> ScanBook`



## bookclerk_plugin_abi::generated::ScanLibraryParams

*Struct*

Params of `RemoteLibrary.scanLibrary` (remote library sync).

**Fields:**
- `force: bool` - When true, force a full rescan even if the guest would otherwise

**Traits:** Eq

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ScanLibraryParams`
- **Default**
  - `fn default() -> ScanLibraryParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanLibraryParams) -> bool`



## bookclerk_plugin_abi::generated::ScanParams

*Struct*

Params of `ContentSource.scan`. The host injects sealed credentials so the
plugin does not need a private credential store under `pluginDataDir`.

**Fields:**
- `plugin_data_dir: String` - Scoped plugin data directory.
- `accounts: Vec<String>` - Account ids to scan; empty means all scan-enabled accounts.
- `page_size: u32` - Storefront page size; the host always sends an explicit value.
- `import_episodes: bool` - When true, import podcast/episode-style rows.
- `import_plus_titles: bool` - When true, import Plus/catalog entitlement titles.
- `credentials: Vec<AccountCredential>` - Host-loaded credential blobs for the requested accounts.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ScanParams`
- **Default**
  - `fn default() -> ScanParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::ScanSummary

*Struct*

Summary result of `ContentSource.scan`.

**Fields:**
- `accounts: u32` - Number of accounts touched during the scan.
- `books_upserted: u32` - Count of titles the guest expects the host to upsert; may mirror
- `pages: u32` - Number of storefront pages fetched.
- `skipped_disabled: u32` - Accounts skipped because `scanEnabled` was false.
- `books: Vec<ScanBook>` - Titles for the host to upsert. Prefer this over plugin-side DB writes.

**Traits:** Eq

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &ScanSummary) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> ScanSummary`
- **Default**
  - `fn default() -> ScanSummary`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::generated::SearchCatalogParams

*Struct*

Params of `ContentSource.searchCatalog`.

**Fields:**
- `query: String` - Free-text search query.
- `region: String` - Storefront region / marketplace code; empty means the guest default.
- `limit: u32` - Maximum hits to return; the host always sends an explicit value.
- `page: u32` - 1-based page for storefronts that page.
- `sort: CatalogSort` - Sort order.
- `field: CatalogField` - Facet restriction.
- `language: Option<String>` - Preferred content language (soft-prioritize; e.g. `en`).

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &SearchCatalogParams) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> SearchCatalogParams`
- **Default**
  - `fn default() -> SearchCatalogParams`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_plugin_abi::generated::SourceAccount

*Struct*

Source account metadata returned from login and stored by the host.

**Fields:**
- `account_id: String` - Stable account id within this source plugin.
- `source: String` - Source plugin id (the host forces this to the guest's install id).
- `marketplace: String` - Storefront marketplace / region code (for example `us`, `uk`).
- `label: Option<String>` - Operator-facing label.
- `scan_enabled: bool` - When true, bare / scheduled scans include this account. Explicit CLI

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SourceAccount`
- **Default**
  - `fn default() -> SourceAccount`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &SourceAccount) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::SourceAccounts

*Struct*

Success payload of `ContentSource.listAccounts`.

**Fields:**
- `accounts: Vec<SourceAccount>` - Accounts the guest knows about.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SourceAccounts`
- **Default**
  - `fn default() -> SourceAccounts`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &SourceAccounts) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::SyncListeningResult

*Struct*

Success payload of `RemoteLibrary.syncListening`.

**Fields:**
- `items: Vec<ListeningProgress>` - Progress snapshots to upsert.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> SyncListeningResult`
- **Default**
  - `fn default() -> SyncListeningResult`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &SyncListeningResult) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



## bookclerk_plugin_abi::generated::read_account_credential

*Function*

Decode a [`AccountCredential`](generated.md#bookclerk_plugin_abigeneratedaccountcredential) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_account_credential(r: plugin_capnp::account_credential::Reader) -> capnp::Result<AccountCredential>
```



## bookclerk_plugin_abi::generated::read_any_message

*Function*

Decodes `bytes` as an unpacked single-segment Cap'n Proto message whose
root is the struct `selector` names, through that struct's `read_*`
codec, and drops the result.

Harness entry point for the `abi_decode` fuzz target and its corpus
replay: every generated decoder runs with a traversal budget derived
from the input size and a fixed nesting limit, so a small buffer cannot
expand into a large graph.

# Errors

Returns the Cap'n Proto error the framing or the codec reports, or a
failure for a selector at or past [`READ_ANY_COUNT`](generated.md#bookclerk_plugin_abigeneratedread_any_count).

```rust
fn read_any_message(selector: u8, bytes: &[u8]) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::read_authenticate_user_params

*Function*

Decode a [`AuthenticateUserParams`](generated.md#bookclerk_plugin_abigeneratedauthenticateuserparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_authenticate_user_params(r: plugin_capnp::authenticate_user_params::Reader) -> capnp::Result<AuthenticateUserParams>
```



## bookclerk_plugin_abi::generated::read_brand

*Function*

Decode a [`Brand`](generated.md#bookclerk_plugin_abigeneratedbrand) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_brand(r: plugin_capnp::brand::Reader) -> capnp::Result<Brand>
```



## bookclerk_plugin_abi::generated::read_catalog_detail

*Function*

Decode a [`CatalogDetail`](generated.md#bookclerk_plugin_abigeneratedcatalogdetail) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_catalog_detail(r: plugin_capnp::catalog_detail::Reader) -> capnp::Result<CatalogDetail>
```



## bookclerk_plugin_abi::generated::read_catalog_detail_params

*Function*

Decode a [`CatalogDetailParams`](generated.md#bookclerk_plugin_abigeneratedcatalogdetailparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_catalog_detail_params(r: plugin_capnp::catalog_detail_params::Reader) -> capnp::Result<CatalogDetailParams>
```



## bookclerk_plugin_abi::generated::read_catalog_detail_reply

*Function*

Decode a `CatalogDetailReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_catalog_detail_reply(r: plugin_capnp::catalog_detail_reply::Reader) -> capnp::Result<Result<CatalogDetail, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_catalog_hit

*Function*

Decode a [`CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_catalog_hit(r: plugin_capnp::catalog_hit::Reader) -> capnp::Result<CatalogHit>
```



## bookclerk_plugin_abi::generated::read_catalog_hits

*Function*

Decode a [`CatalogHits`](generated.md#bookclerk_plugin_abigeneratedcataloghits) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_catalog_hits(r: plugin_capnp::catalog_hits::Reader) -> capnp::Result<CatalogHits>
```



## bookclerk_plugin_abi::generated::read_catalog_hits_reply

*Function*

Decode a `CatalogHitsReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_catalog_hits_reply(r: plugin_capnp::catalog_hits_reply::Reader) -> capnp::Result<Result<CatalogHits, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_chapter_marker

*Function*

Decode a [`ChapterMarker`](generated.md#bookclerk_plugin_abigeneratedchaptermarker) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_chapter_marker(r: plugin_capnp::chapter_marker::Reader) -> capnp::Result<ChapterMarker>
```



## bookclerk_plugin_abi::generated::read_cli_arg

*Function*

Decode a [`CliArg`](generated.md#bookclerk_plugin_abigeneratedcliarg) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_arg(r: plugin_capnp::cli_arg::Reader) -> capnp::Result<CliArg>
```



## bookclerk_plugin_abi::generated::read_cli_arg_spec

*Function*

Decode a [`CliArgSpec`](generated.md#bookclerk_plugin_abigeneratedcliargspec) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_arg_spec(r: plugin_capnp::cli_arg_spec::Reader) -> capnp::Result<CliArgSpec>
```



## bookclerk_plugin_abi::generated::read_cli_command_spec

*Function*

Decode a [`CliCommandSpec`](generated.md#bookclerk_plugin_abigeneratedclicommandspec) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_command_spec(r: plugin_capnp::cli_command_spec::Reader) -> capnp::Result<CliCommandSpec>
```



## bookclerk_plugin_abi::generated::read_cli_invoke_params

*Function*

Decode a [`CliInvokeParams`](generated.md#bookclerk_plugin_abigeneratedcliinvokeparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_invoke_params(r: plugin_capnp::cli_invoke_params::Reader) -> capnp::Result<CliInvokeParams>
```



## bookclerk_plugin_abi::generated::read_cli_invoke_reply

*Function*

Decode a `CliInvokeReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_cli_invoke_reply(r: plugin_capnp::cli_invoke_reply::Reader) -> capnp::Result<Result<CliInvokeResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_cli_invoke_result

*Function*

Decode a [`CliInvokeResult`](generated.md#bookclerk_plugin_abigeneratedcliinvokeresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_invoke_result(r: plugin_capnp::cli_invoke_result::Reader) -> capnp::Result<CliInvokeResult>
```



## bookclerk_plugin_abi::generated::read_cli_schema

*Function*

Decode a [`CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_cli_schema(r: plugin_capnp::cli_schema::Reader) -> capnp::Result<CliSchema>
```



## bookclerk_plugin_abi::generated::read_cli_schema_reply

*Function*

Decode a `CliSchemaReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_cli_schema_reply(r: plugin_capnp::cli_schema_reply::Reader) -> capnp::Result<Result<CliSchema, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_config_option

*Function*

Decode a [`ConfigOption`](generated.md#bookclerk_plugin_abigeneratedconfigoption) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_config_option(r: plugin_capnp::config_option::Reader) -> capnp::Result<ConfigOption>
```



## bookclerk_plugin_abi::generated::read_config_option_value

*Function*

Decode a [`ConfigOptionValue`](generated.md#bookclerk_plugin_abigeneratedconfigoptionvalue) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_config_option_value(r: plugin_capnp::config_option_value::Reader) -> capnp::Result<ConfigOptionValue>
```



## bookclerk_plugin_abi::generated::read_database_adapter_config

*Function*

Decode a [`DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_database_adapter_config(r: plugin_capnp::database_adapter_config::Reader) -> capnp::Result<DatabaseAdapterConfig>
```



## bookclerk_plugin_abi::generated::read_diagnose_reply

*Function*

Decode a `DiagnoseReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_diagnose_reply(r: plugin_capnp::diagnose_reply::Reader) -> capnp::Result<Result<DiagnoseResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_diagnose_result

*Function*

Decode a [`DiagnoseResult`](generated.md#bookclerk_plugin_abigenerateddiagnoseresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_diagnose_result(r: plugin_capnp::diagnose_result::Reader) -> capnp::Result<DiagnoseResult>
```



## bookclerk_plugin_abi::generated::read_event_consumer_spec

*Function*

Decode a [`EventConsumerSpec`](generated.md#bookclerk_plugin_abigeneratedeventconsumerspec) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_event_consumer_spec(r: plugin_capnp::event_consumer_spec::Reader) -> capnp::Result<EventConsumerSpec>
```



## bookclerk_plugin_abi::generated::read_event_poll_reply

*Function*

Decode a `EventPollReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_event_poll_reply(r: plugin_capnp::event_poll_reply::Reader) -> capnp::Result<Result<EventPollResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_event_poll_result

*Function*

Decode a [`EventPollResult`](generated.md#bookclerk_plugin_abigeneratedeventpollresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_event_poll_result(r: plugin_capnp::event_poll_result::Reader) -> capnp::Result<EventPollResult>
```



## bookclerk_plugin_abi::generated::read_expand_candidates_params

*Function*

Decode a [`ExpandCandidatesParams`](generated.md#bookclerk_plugin_abigeneratedexpandcandidatesparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_expand_candidates_params(r: plugin_capnp::expand_candidates_params::Reader) -> capnp::Result<ExpandCandidatesParams>
```



## bookclerk_plugin_abi::generated::read_external_user

*Function*

Decode a [`ExternalUser`](generated.md#bookclerk_plugin_abigeneratedexternaluser) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_external_user(r: plugin_capnp::external_user::Reader) -> capnp::Result<ExternalUser>
```



## bookclerk_plugin_abi::generated::read_external_user_reply

*Function*

Decode a `ExternalUserReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_external_user_reply(r: plugin_capnp::external_user_reply::Reader) -> capnp::Result<Result<ExternalUser, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_fetch_options

*Function*

Decode a [`FetchOptions`](generated.md#bookclerk_plugin_abigeneratedfetchoptions) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_fetch_options(r: plugin_capnp::fetch_options::Reader) -> capnp::Result<FetchOptions>
```



## bookclerk_plugin_abi::generated::read_fetch_title_params

*Function*

Decode a [`FetchTitleParams`](generated.md#bookclerk_plugin_abigeneratedfetchtitleparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_fetch_title_params(r: plugin_capnp::fetch_title_params::Reader) -> capnp::Result<FetchTitleParams>
```



## bookclerk_plugin_abi::generated::read_fetch_title_reply

*Function*

Decode a `FetchTitleReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_fetch_title_reply(r: plugin_capnp::fetch_title_reply::Reader) -> capnp::Result<Result<PlainFetch, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_invocation

*Function*

Decode a [`Invocation`](generated.md#bookclerk_plugin_abigeneratedinvocation) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_invocation(r: plugin_capnp::invocation::Reader) -> capnp::Result<Invocation>
```



## bookclerk_plugin_abi::generated::read_kv_delete_ok

*Function*

Decode a [`KvDeleteOk`](generated.md#bookclerk_plugin_abigeneratedkvdeleteok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_delete_ok(r: plugin_capnp::kv_delete_ok::Reader) -> capnp::Result<KvDeleteOk>
```



## bookclerk_plugin_abi::generated::read_kv_delete_reply

*Function*

Decode a `KvDeleteReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_kv_delete_reply(r: plugin_capnp::kv_delete_reply::Reader) -> capnp::Result<Result<KvDeleteOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_kv_get_ok

*Function*

Decode a [`KvGetOk`](generated.md#bookclerk_plugin_abigeneratedkvgetok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_get_ok(r: plugin_capnp::kv_get_ok::Reader) -> capnp::Result<KvGetOk>
```



## bookclerk_plugin_abi::generated::read_kv_get_reply

*Function*

Decode a `KvGetReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_kv_get_reply(r: plugin_capnp::kv_get_reply::Reader) -> capnp::Result<Result<KvGetOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_kv_list_key

*Function*

Decode a [`KvListKey`](generated.md#bookclerk_plugin_abigeneratedkvlistkey) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_list_key(r: plugin_capnp::kv_list_key::Reader) -> capnp::Result<KvListKey>
```



## bookclerk_plugin_abi::generated::read_kv_list_ok

*Function*

Decode a [`KvListOk`](generated.md#bookclerk_plugin_abigeneratedkvlistok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_list_ok(r: plugin_capnp::kv_list_ok::Reader) -> capnp::Result<KvListOk>
```



## bookclerk_plugin_abi::generated::read_kv_list_options

*Function*

Decode a [`KvListOptions`](generated.md#bookclerk_plugin_abigeneratedkvlistoptions) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_list_options(r: plugin_capnp::kv_list_options::Reader) -> capnp::Result<KvListOptions>
```



## bookclerk_plugin_abi::generated::read_kv_list_reply

*Function*

Decode a `KvListReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_kv_list_reply(r: plugin_capnp::kv_list_reply::Reader) -> capnp::Result<Result<KvListOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_kv_put_ok

*Function*

Decode a [`KvPutOk`](generated.md#bookclerk_plugin_abigeneratedkvputok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_put_ok(r: plugin_capnp::kv_put_ok::Reader) -> capnp::Result<KvPutOk>
```



## bookclerk_plugin_abi::generated::read_kv_put_options

*Function*

Decode a [`KvPutOptions`](generated.md#bookclerk_plugin_abigeneratedkvputoptions) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_kv_put_options(r: plugin_capnp::kv_put_options::Reader) -> capnp::Result<KvPutOptions>
```



## bookclerk_plugin_abi::generated::read_kv_put_reply

*Function*

Decode a `KvPutReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_kv_put_reply(r: plugin_capnp::kv_put_reply::Reader) -> capnp::Result<Result<KvPutOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_list_deals_params

*Function*

Decode a [`ListDealsParams`](generated.md#bookclerk_plugin_abigeneratedlistdealsparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_list_deals_params(r: plugin_capnp::list_deals_params::Reader) -> capnp::Result<ListDealsParams>
```



## bookclerk_plugin_abi::generated::read_listening_progress

*Function*

Decode a [`ListeningProgress`](generated.md#bookclerk_plugin_abigeneratedlisteningprogress) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_listening_progress(r: plugin_capnp::listening_progress::Reader) -> capnp::Result<ListeningProgress>
```



## bookclerk_plugin_abi::generated::read_login_complete_params

*Function*

Decode a [`LoginCompleteParams`](generated.md#bookclerk_plugin_abigeneratedlogincompleteparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_login_complete_params(r: plugin_capnp::login_complete_params::Reader) -> capnp::Result<LoginCompleteParams>
```



## bookclerk_plugin_abi::generated::read_login_params

*Function*

Decode a [`LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_login_params(r: plugin_capnp::login_params::Reader) -> capnp::Result<LoginParams>
```



## bookclerk_plugin_abi::generated::read_login_reply

*Function*

Decode a `LoginReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_login_reply(r: plugin_capnp::login_reply::Reader) -> capnp::Result<Result<LoginResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_login_result

*Function*

Decode a [`LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_login_result(r: plugin_capnp::login_result::Reader) -> capnp::Result<LoginResult>
```



## bookclerk_plugin_abi::generated::read_login_start_reply

*Function*

Decode a `LoginStartReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_login_start_reply(r: plugin_capnp::login_start_reply::Reader) -> capnp::Result<Result<LoginStartResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_login_start_result

*Function*

Decode a [`LoginStartResult`](generated.md#bookclerk_plugin_abigeneratedloginstartresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_login_start_result(r: plugin_capnp::login_start_result::Reader) -> capnp::Result<LoginStartResult>
```



## bookclerk_plugin_abi::generated::read_plain_fetch

*Function*

Decode a [`PlainFetch`](generated.md#bookclerk_plugin_abigeneratedplainfetch) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_plain_fetch(r: plugin_capnp::plain_fetch::Reader) -> capnp::Result<PlainFetch>
```



## bookclerk_plugin_abi::generated::read_plain_part

*Function*

Decode a [`PlainPart`](generated.md#bookclerk_plugin_abigeneratedplainpart) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_plain_part(r: plugin_capnp::plain_part::Reader) -> capnp::Result<PlainPart>
```



## bookclerk_plugin_abi::generated::read_plugin_capabilities

*Function*

Decode a [`PluginCapabilities`](generated.md#bookclerk_plugin_abigeneratedplugincapabilities) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_plugin_capabilities(r: plugin_capnp::plugin_capabilities::Reader) -> capnp::Result<PluginCapabilities>
```



## bookclerk_plugin_abi::generated::read_plugin_event

*Function*

Decode a [`PluginEvent`](generated.md#bookclerk_plugin_abigeneratedpluginevent) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_plugin_event(r: plugin_capnp::plugin_event::Reader) -> capnp::Result<PluginEvent>
```



## bookclerk_plugin_abi::generated::read_publish_ok

*Function*

Decode a [`PublishOk`](generated.md#bookclerk_plugin_abigeneratedpublishok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_publish_ok(r: plugin_capnp::publish_ok::Reader) -> capnp::Result<PublishOk>
```



## bookclerk_plugin_abi::generated::read_publish_reply

*Function*

Decode a `PublishReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_publish_reply(r: plugin_capnp::publish_reply::Reader) -> capnp::Result<Result<PublishOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_purchase_hint

*Function*

Decode a [`PurchaseHint`](generated.md#bookclerk_plugin_abigeneratedpurchasehint) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_purchase_hint(r: plugin_capnp::purchase_hint::Reader) -> capnp::Result<PurchaseHint>
```



## bookclerk_plugin_abi::generated::read_purchase_hint_params

*Function*

Decode a [`PurchaseHintParams`](generated.md#bookclerk_plugin_abigeneratedpurchasehintparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_purchase_hint_params(r: plugin_capnp::purchase_hint_params::Reader) -> capnp::Result<PurchaseHintParams>
```



## bookclerk_plugin_abi::generated::read_purchase_hint_reply

*Function*

Decode a `PurchaseHintReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_purchase_hint_reply(r: plugin_capnp::purchase_hint_reply::Reader) -> capnp::Result<Result<PurchaseHintResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_purchase_hint_result

*Function*

Decode a [`PurchaseHintResult`](generated.md#bookclerk_plugin_abigeneratedpurchasehintresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_purchase_hint_result(r: plugin_capnp::purchase_hint_result::Reader) -> capnp::Result<PurchaseHintResult>
```



## bookclerk_plugin_abi::generated::read_queue_batch

*Function*

Decode a [`QueueBatch`](generated.md#bookclerk_plugin_abigeneratedqueuebatch) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_batch(r: plugin_capnp::queue_batch::Reader) -> capnp::Result<QueueBatch>
```



## bookclerk_plugin_abi::generated::read_queue_batch_ok

*Function*

Decode a [`QueueBatchOk`](generated.md#bookclerk_plugin_abigeneratedqueuebatchok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_batch_ok(r: plugin_capnp::queue_batch_ok::Reader) -> capnp::Result<QueueBatchOk>
```



## bookclerk_plugin_abi::generated::read_queue_batch_reply

*Function*

Decode a `QueueBatchReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_queue_batch_reply(r: plugin_capnp::queue_batch_reply::Reader) -> capnp::Result<Result<QueueBatchOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_queue_message

*Function*

Decode a [`QueueMessage`](generated.md#bookclerk_plugin_abigeneratedqueuemessage) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_message(r: plugin_capnp::queue_message::Reader) -> capnp::Result<QueueMessage>
```



## bookclerk_plugin_abi::generated::read_queue_result

*Function*

Decode a `QueueResult` union.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_queue_result(r: plugin_capnp::queue_result::Reader) -> capnp::Result<QueueResult>
```



## bookclerk_plugin_abi::generated::read_queue_retry

*Function*

Decode a [`QueueRetry`](generated.md#bookclerk_plugin_abigeneratedqueueretry) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_retry(r: plugin_capnp::queue_retry::Reader) -> capnp::Result<QueueRetry>
```



## bookclerk_plugin_abi::generated::read_queue_send_message

*Function*

Decode a [`QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_send_message(r: plugin_capnp::queue_send_message::Reader) -> capnp::Result<QueueSendMessage>
```



## bookclerk_plugin_abi::generated::read_queue_send_ok

*Function*

Decode a [`QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_queue_send_ok(r: plugin_capnp::queue_send_ok::Reader) -> capnp::Result<QueueSendOk>
```



## bookclerk_plugin_abi::generated::read_queue_send_reply

*Function*

Decode a `QueueSendReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_queue_send_reply(r: plugin_capnp::queue_send_reply::Reader) -> capnp::Result<Result<QueueSendOk, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_scan_book

*Function*

Decode a [`ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_scan_book(r: plugin_capnp::scan_book::Reader) -> capnp::Result<ScanBook>
```



## bookclerk_plugin_abi::generated::read_scan_library_params

*Function*

Decode a [`ScanLibraryParams`](generated.md#bookclerk_plugin_abigeneratedscanlibraryparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_scan_library_params(r: plugin_capnp::scan_library_params::Reader) -> capnp::Result<ScanLibraryParams>
```



## bookclerk_plugin_abi::generated::read_scan_params

*Function*

Decode a [`ScanParams`](generated.md#bookclerk_plugin_abigeneratedscanparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_scan_params(r: plugin_capnp::scan_params::Reader) -> capnp::Result<ScanParams>
```



## bookclerk_plugin_abi::generated::read_scan_reply

*Function*

Decode a `ScanReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_scan_reply(r: plugin_capnp::scan_reply::Reader) -> capnp::Result<Result<ScanSummary, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_scan_summary

*Function*

Decode a [`ScanSummary`](generated.md#bookclerk_plugin_abigeneratedscansummary) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_scan_summary(r: plugin_capnp::scan_summary::Reader) -> capnp::Result<ScanSummary>
```



## bookclerk_plugin_abi::generated::read_search_catalog_params

*Function*

Decode a [`SearchCatalogParams`](generated.md#bookclerk_plugin_abigeneratedsearchcatalogparams) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_search_catalog_params(r: plugin_capnp::search_catalog_params::Reader) -> capnp::Result<SearchCatalogParams>
```



## bookclerk_plugin_abi::generated::read_source_account

*Function*

Decode a [`SourceAccount`](generated.md#bookclerk_plugin_abigeneratedsourceaccount) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_source_account(r: plugin_capnp::source_account::Reader) -> capnp::Result<SourceAccount>
```



## bookclerk_plugin_abi::generated::read_source_accounts

*Function*

Decode a [`SourceAccounts`](generated.md#bookclerk_plugin_abigeneratedsourceaccounts) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_source_accounts(r: plugin_capnp::source_accounts::Reader) -> capnp::Result<SourceAccounts>
```



## bookclerk_plugin_abi::generated::read_source_accounts_reply

*Function*

Decode a `SourceAccountsReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_source_accounts_reply(r: plugin_capnp::source_accounts_reply::Reader) -> capnp::Result<Result<SourceAccounts, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_sync_listening_reply

*Function*

Decode a `SyncListeningReply` union into a typed result.

# Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

```rust
fn read_sync_listening_reply(r: plugin_capnp::sync_listening_reply::Reader) -> capnp::Result<Result<SyncListeningResult, crate::PluginError>>
```



## bookclerk_plugin_abi::generated::read_sync_listening_result

*Function*

Decode a [`SyncListeningResult`](generated.md#bookclerk_plugin_abigeneratedsynclisteningresult) from a Cap'n Proto reader.

# Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

```rust
fn read_sync_listening_result(r: plugin_capnp::sync_listening_result::Reader) -> capnp::Result<SyncListeningResult>
```



## bookclerk_plugin_abi::generated::roundtrip_any_message

*Function*

Decodes `bytes` as the struct `selector` names (as [`read_any_message`](generated.md#bookclerk_plugin_abigeneratedread_any_message)
does), writes the value back through that struct's `write_*` codec,
decodes the result and requires it to equal the first value.

Harness entry point for the `abi_roundtrip` fuzz target and its corpus
replay. The first decode runs under the same traversal and nesting
budget as [`read_any_message`](generated.md#bookclerk_plugin_abigeneratedread_any_message); the re-decode runs on bytes this crate
produced from an already bounded value, so it uses the default budget.
Equality is the DTO's `PartialEq`; because `NaN != NaN`, two values
that `PartialEq` separates still count as equal when their `Debug`
renderings agree (a field-by-field comparison that prints every NaN
the same way and keeps `-0.0` apart from `0.0`) and the re-decoded
value re-encodes to the same bytes (which also covers fields whose
`Debug` redacts).

# Errors

`RoundTripFailure::Rejected` when the input does not decode or the
selector is at or past [`READ_ANY_COUNT`](generated.md#bookclerk_plugin_abigeneratedread_any_count); `RoundTripFailure::Codec`
when the decoded value does not survive `write_*` then `read_*` intact.

```rust
fn roundtrip_any_message(selector: u8, bytes: &[u8]) -> Result<(), RoundTripFailure>
```



## bookclerk_plugin_abi::generated::write_account_credential

*Function*

Encode a [`AccountCredential`](generated.md#bookclerk_plugin_abigeneratedaccountcredential) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_account_credential(b: plugin_capnp::account_credential::Builder, v: &AccountCredential) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_authenticate_user_params

*Function*

Encode a [`AuthenticateUserParams`](generated.md#bookclerk_plugin_abigeneratedauthenticateuserparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_authenticate_user_params(b: plugin_capnp::authenticate_user_params::Builder, v: &AuthenticateUserParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_brand

*Function*

Encode a [`Brand`](generated.md#bookclerk_plugin_abigeneratedbrand) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_brand(b: plugin_capnp::brand::Builder, v: &Brand) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_detail

*Function*

Encode a [`CatalogDetail`](generated.md#bookclerk_plugin_abigeneratedcatalogdetail) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_catalog_detail(b: plugin_capnp::catalog_detail::Builder, v: &CatalogDetail) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_detail_params

*Function*

Encode a [`CatalogDetailParams`](generated.md#bookclerk_plugin_abigeneratedcatalogdetailparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_catalog_detail_params(b: plugin_capnp::catalog_detail_params::Builder, v: &CatalogDetailParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_detail_reply

*Function*

Encode a `CatalogDetailReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_catalog_detail_reply(b: plugin_capnp::catalog_detail_reply::Builder, v: &Result<CatalogDetail, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_hit

*Function*

Encode a [`CatalogHit`](generated.md#bookclerk_plugin_abigeneratedcataloghit) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_catalog_hit(b: plugin_capnp::catalog_hit::Builder, v: &CatalogHit) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_hits

*Function*

Encode a [`CatalogHits`](generated.md#bookclerk_plugin_abigeneratedcataloghits) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_catalog_hits(b: plugin_capnp::catalog_hits::Builder, v: &CatalogHits) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_catalog_hits_reply

*Function*

Encode a `CatalogHitsReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_catalog_hits_reply(b: plugin_capnp::catalog_hits_reply::Builder, v: &Result<CatalogHits, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_chapter_marker

*Function*

Encode a [`ChapterMarker`](generated.md#bookclerk_plugin_abigeneratedchaptermarker) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_chapter_marker(b: plugin_capnp::chapter_marker::Builder, v: &ChapterMarker) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_arg

*Function*

Encode a [`CliArg`](generated.md#bookclerk_plugin_abigeneratedcliarg) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_arg(b: plugin_capnp::cli_arg::Builder, v: &CliArg) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_arg_spec

*Function*

Encode a [`CliArgSpec`](generated.md#bookclerk_plugin_abigeneratedcliargspec) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_arg_spec(b: plugin_capnp::cli_arg_spec::Builder, v: &CliArgSpec) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_command_spec

*Function*

Encode a [`CliCommandSpec`](generated.md#bookclerk_plugin_abigeneratedclicommandspec) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_command_spec(b: plugin_capnp::cli_command_spec::Builder, v: &CliCommandSpec) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_invoke_params

*Function*

Encode a [`CliInvokeParams`](generated.md#bookclerk_plugin_abigeneratedcliinvokeparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_invoke_params(b: plugin_capnp::cli_invoke_params::Builder, v: &CliInvokeParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_invoke_reply

*Function*

Encode a `CliInvokeReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_cli_invoke_reply(b: plugin_capnp::cli_invoke_reply::Builder, v: &Result<CliInvokeResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_invoke_result

*Function*

Encode a [`CliInvokeResult`](generated.md#bookclerk_plugin_abigeneratedcliinvokeresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_invoke_result(b: plugin_capnp::cli_invoke_result::Builder, v: &CliInvokeResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_schema

*Function*

Encode a [`CliSchema`](generated.md#bookclerk_plugin_abigeneratedclischema) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_cli_schema(b: plugin_capnp::cli_schema::Builder, v: &CliSchema) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_cli_schema_reply

*Function*

Encode a `CliSchemaReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_cli_schema_reply(b: plugin_capnp::cli_schema_reply::Builder, v: &Result<CliSchema, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_config_option

*Function*

Encode a [`ConfigOption`](generated.md#bookclerk_plugin_abigeneratedconfigoption) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_config_option(b: plugin_capnp::config_option::Builder, v: &ConfigOption) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_config_option_value

*Function*

Encode a [`ConfigOptionValue`](generated.md#bookclerk_plugin_abigeneratedconfigoptionvalue) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_config_option_value(b: plugin_capnp::config_option_value::Builder, v: &ConfigOptionValue) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_database_adapter_config

*Function*

Encode a [`DatabaseAdapterConfig`](generated.md#bookclerk_plugin_abigenerateddatabaseadapterconfig) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_database_adapter_config(b: plugin_capnp::database_adapter_config::Builder, v: &DatabaseAdapterConfig) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_diagnose_reply

*Function*

Encode a `DiagnoseReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_diagnose_reply(b: plugin_capnp::diagnose_reply::Builder, v: &Result<DiagnoseResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_diagnose_result

*Function*

Encode a [`DiagnoseResult`](generated.md#bookclerk_plugin_abigenerateddiagnoseresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_diagnose_result(b: plugin_capnp::diagnose_result::Builder, v: &DiagnoseResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_event_consumer_spec

*Function*

Encode a [`EventConsumerSpec`](generated.md#bookclerk_plugin_abigeneratedeventconsumerspec) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_event_consumer_spec(b: plugin_capnp::event_consumer_spec::Builder, v: &EventConsumerSpec) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_event_poll_reply

*Function*

Encode a `EventPollReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_event_poll_reply(b: plugin_capnp::event_poll_reply::Builder, v: &Result<EventPollResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_event_poll_result

*Function*

Encode a [`EventPollResult`](generated.md#bookclerk_plugin_abigeneratedeventpollresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_event_poll_result(b: plugin_capnp::event_poll_result::Builder, v: &EventPollResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_expand_candidates_params

*Function*

Encode a [`ExpandCandidatesParams`](generated.md#bookclerk_plugin_abigeneratedexpandcandidatesparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_expand_candidates_params(b: plugin_capnp::expand_candidates_params::Builder, v: &ExpandCandidatesParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_external_user

*Function*

Encode a [`ExternalUser`](generated.md#bookclerk_plugin_abigeneratedexternaluser) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_external_user(b: plugin_capnp::external_user::Builder, v: &ExternalUser) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_external_user_reply

*Function*

Encode a `ExternalUserReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_external_user_reply(b: plugin_capnp::external_user_reply::Builder, v: &Result<ExternalUser, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_fetch_options

*Function*

Encode a [`FetchOptions`](generated.md#bookclerk_plugin_abigeneratedfetchoptions) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_fetch_options(b: plugin_capnp::fetch_options::Builder, v: &FetchOptions) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_fetch_title_params

*Function*

Encode a [`FetchTitleParams`](generated.md#bookclerk_plugin_abigeneratedfetchtitleparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_fetch_title_params(b: plugin_capnp::fetch_title_params::Builder, v: &FetchTitleParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_fetch_title_reply

*Function*

Encode a `FetchTitleReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_fetch_title_reply(b: plugin_capnp::fetch_title_reply::Builder, v: &Result<PlainFetch, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_invocation

*Function*

Encode a [`Invocation`](generated.md#bookclerk_plugin_abigeneratedinvocation) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_invocation(b: plugin_capnp::invocation::Builder, v: &Invocation) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_delete_ok

*Function*

Encode a [`KvDeleteOk`](generated.md#bookclerk_plugin_abigeneratedkvdeleteok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_delete_ok(b: plugin_capnp::kv_delete_ok::Builder, v: &KvDeleteOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_delete_reply

*Function*

Encode a `KvDeleteReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_kv_delete_reply(b: plugin_capnp::kv_delete_reply::Builder, v: &Result<KvDeleteOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_get_ok

*Function*

Encode a [`KvGetOk`](generated.md#bookclerk_plugin_abigeneratedkvgetok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_get_ok(b: plugin_capnp::kv_get_ok::Builder, v: &KvGetOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_get_reply

*Function*

Encode a `KvGetReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_kv_get_reply(b: plugin_capnp::kv_get_reply::Builder, v: &Result<KvGetOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_list_key

*Function*

Encode a [`KvListKey`](generated.md#bookclerk_plugin_abigeneratedkvlistkey) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_list_key(b: plugin_capnp::kv_list_key::Builder, v: &KvListKey) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_list_ok

*Function*

Encode a [`KvListOk`](generated.md#bookclerk_plugin_abigeneratedkvlistok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_list_ok(b: plugin_capnp::kv_list_ok::Builder, v: &KvListOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_list_options

*Function*

Encode a [`KvListOptions`](generated.md#bookclerk_plugin_abigeneratedkvlistoptions) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_list_options(b: plugin_capnp::kv_list_options::Builder, v: &KvListOptions) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_list_reply

*Function*

Encode a `KvListReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_kv_list_reply(b: plugin_capnp::kv_list_reply::Builder, v: &Result<KvListOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_put_ok

*Function*

Encode a [`KvPutOk`](generated.md#bookclerk_plugin_abigeneratedkvputok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_put_ok(b: plugin_capnp::kv_put_ok::Builder, v: &KvPutOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_put_options

*Function*

Encode a [`KvPutOptions`](generated.md#bookclerk_plugin_abigeneratedkvputoptions) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_kv_put_options(b: plugin_capnp::kv_put_options::Builder, v: &KvPutOptions) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_kv_put_reply

*Function*

Encode a `KvPutReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_kv_put_reply(b: plugin_capnp::kv_put_reply::Builder, v: &Result<KvPutOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_list_deals_params

*Function*

Encode a [`ListDealsParams`](generated.md#bookclerk_plugin_abigeneratedlistdealsparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_list_deals_params(b: plugin_capnp::list_deals_params::Builder, v: &ListDealsParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_listening_progress

*Function*

Encode a [`ListeningProgress`](generated.md#bookclerk_plugin_abigeneratedlisteningprogress) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_listening_progress(b: plugin_capnp::listening_progress::Builder, v: &ListeningProgress) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_complete_params

*Function*

Encode a [`LoginCompleteParams`](generated.md#bookclerk_plugin_abigeneratedlogincompleteparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_login_complete_params(b: plugin_capnp::login_complete_params::Builder, v: &LoginCompleteParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_params

*Function*

Encode a [`LoginParams`](generated.md#bookclerk_plugin_abigeneratedloginparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_login_params(b: plugin_capnp::login_params::Builder, v: &LoginParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_reply

*Function*

Encode a `LoginReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_login_reply(b: plugin_capnp::login_reply::Builder, v: &Result<LoginResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_result

*Function*

Encode a [`LoginResult`](generated.md#bookclerk_plugin_abigeneratedloginresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_login_result(b: plugin_capnp::login_result::Builder, v: &LoginResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_start_reply

*Function*

Encode a `LoginStartReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_login_start_reply(b: plugin_capnp::login_start_reply::Builder, v: &Result<LoginStartResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_login_start_result

*Function*

Encode a [`LoginStartResult`](generated.md#bookclerk_plugin_abigeneratedloginstartresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_login_start_result(b: plugin_capnp::login_start_result::Builder, v: &LoginStartResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_plain_fetch

*Function*

Encode a [`PlainFetch`](generated.md#bookclerk_plugin_abigeneratedplainfetch) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_plain_fetch(b: plugin_capnp::plain_fetch::Builder, v: &PlainFetch) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_plain_part

*Function*

Encode a [`PlainPart`](generated.md#bookclerk_plugin_abigeneratedplainpart) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_plain_part(b: plugin_capnp::plain_part::Builder, v: &PlainPart) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_plugin_capabilities

*Function*

Encode a [`PluginCapabilities`](generated.md#bookclerk_plugin_abigeneratedplugincapabilities) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_plugin_capabilities(b: plugin_capnp::plugin_capabilities::Builder, v: &PluginCapabilities) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_plugin_event

*Function*

Encode a [`PluginEvent`](generated.md#bookclerk_plugin_abigeneratedpluginevent) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_plugin_event(b: plugin_capnp::plugin_event::Builder, v: &PluginEvent) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_publish_ok

*Function*

Encode a [`PublishOk`](generated.md#bookclerk_plugin_abigeneratedpublishok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_publish_ok(b: plugin_capnp::publish_ok::Builder, v: &PublishOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_publish_reply

*Function*

Encode a `PublishReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_publish_reply(b: plugin_capnp::publish_reply::Builder, v: &Result<PublishOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_purchase_hint

*Function*

Encode a [`PurchaseHint`](generated.md#bookclerk_plugin_abigeneratedpurchasehint) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_purchase_hint(b: plugin_capnp::purchase_hint::Builder, v: &PurchaseHint) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_purchase_hint_params

*Function*

Encode a [`PurchaseHintParams`](generated.md#bookclerk_plugin_abigeneratedpurchasehintparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_purchase_hint_params(b: plugin_capnp::purchase_hint_params::Builder, v: &PurchaseHintParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_purchase_hint_reply

*Function*

Encode a `PurchaseHintReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_purchase_hint_reply(b: plugin_capnp::purchase_hint_reply::Builder, v: &Result<PurchaseHintResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_purchase_hint_result

*Function*

Encode a [`PurchaseHintResult`](generated.md#bookclerk_plugin_abigeneratedpurchasehintresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_purchase_hint_result(b: plugin_capnp::purchase_hint_result::Builder, v: &PurchaseHintResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_batch

*Function*

Encode a [`QueueBatch`](generated.md#bookclerk_plugin_abigeneratedqueuebatch) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_batch(b: plugin_capnp::queue_batch::Builder, v: &QueueBatch) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_batch_ok

*Function*

Encode a [`QueueBatchOk`](generated.md#bookclerk_plugin_abigeneratedqueuebatchok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_batch_ok(b: plugin_capnp::queue_batch_ok::Builder, v: &QueueBatchOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_batch_reply

*Function*

Encode a `QueueBatchReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_queue_batch_reply(b: plugin_capnp::queue_batch_reply::Builder, v: &Result<QueueBatchOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_message

*Function*

Encode a [`QueueMessage`](generated.md#bookclerk_plugin_abigeneratedqueuemessage) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_message(b: plugin_capnp::queue_message::Builder, v: &QueueMessage) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_result

*Function*

Encode a `QueueResult` union.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_queue_result(b: plugin_capnp::queue_result::Builder, v: &QueueResult) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_retry

*Function*

Encode a [`QueueRetry`](generated.md#bookclerk_plugin_abigeneratedqueueretry) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_retry(b: plugin_capnp::queue_retry::Builder, v: &QueueRetry) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_send_message

*Function*

Encode a [`QueueSendMessage`](generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_send_message(b: plugin_capnp::queue_send_message::Builder, v: &QueueSendMessage) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_send_ok

*Function*

Encode a [`QueueSendOk`](generated.md#bookclerk_plugin_abigeneratedqueuesendok) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_queue_send_ok(b: plugin_capnp::queue_send_ok::Builder, v: &QueueSendOk) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_queue_send_reply

*Function*

Encode a `QueueSendReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_queue_send_reply(b: plugin_capnp::queue_send_reply::Builder, v: &Result<QueueSendOk, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_scan_book

*Function*

Encode a [`ScanBook`](generated.md#bookclerk_plugin_abigeneratedscanbook) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_scan_book(b: plugin_capnp::scan_book::Builder, v: &ScanBook) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_scan_library_params

*Function*

Encode a [`ScanLibraryParams`](generated.md#bookclerk_plugin_abigeneratedscanlibraryparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_scan_library_params(b: plugin_capnp::scan_library_params::Builder, v: &ScanLibraryParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_scan_params

*Function*

Encode a [`ScanParams`](generated.md#bookclerk_plugin_abigeneratedscanparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_scan_params(b: plugin_capnp::scan_params::Builder, v: &ScanParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_scan_reply

*Function*

Encode a `ScanReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_scan_reply(b: plugin_capnp::scan_reply::Builder, v: &Result<ScanSummary, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_scan_summary

*Function*

Encode a [`ScanSummary`](generated.md#bookclerk_plugin_abigeneratedscansummary) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_scan_summary(b: plugin_capnp::scan_summary::Builder, v: &ScanSummary) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_search_catalog_params

*Function*

Encode a [`SearchCatalogParams`](generated.md#bookclerk_plugin_abigeneratedsearchcatalogparams) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_search_catalog_params(b: plugin_capnp::search_catalog_params::Builder, v: &SearchCatalogParams) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_source_account

*Function*

Encode a [`SourceAccount`](generated.md#bookclerk_plugin_abigeneratedsourceaccount) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_source_account(b: plugin_capnp::source_account::Builder, v: &SourceAccount) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_source_accounts

*Function*

Encode a [`SourceAccounts`](generated.md#bookclerk_plugin_abigeneratedsourceaccounts) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_source_accounts(b: plugin_capnp::source_accounts::Builder, v: &SourceAccounts) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_source_accounts_reply

*Function*

Encode a `SourceAccountsReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_source_accounts_reply(b: plugin_capnp::source_accounts_reply::Builder, v: &Result<SourceAccounts, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_sync_listening_reply

*Function*

Encode a `SyncListeningReply` union from a typed result.

# Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

```rust
fn write_sync_listening_reply(b: plugin_capnp::sync_listening_reply::Builder, v: &Result<SyncListeningResult, crate::PluginError>) -> capnp::Result<()>
```



## bookclerk_plugin_abi::generated::write_sync_listening_result

*Function*

Encode a [`SyncListeningResult`](generated.md#bookclerk_plugin_abigeneratedsynclisteningresult) onto a Cap'n Proto builder.

# Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

```rust
fn write_sync_listening_result(b: plugin_capnp::sync_listening_result::Builder, v: &SyncListeningResult) -> capnp::Result<()>
```



