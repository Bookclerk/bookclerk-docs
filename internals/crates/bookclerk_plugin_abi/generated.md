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
| [`CatalogCapabilities`](#catalogcapabilities) | struct | Catalog behaviours a storefront declares in `describe()`. Catalog search reads them instead of special-casing plugin ids; every flag defaults to false, so a guest that says nothing keeps the plain behaviour. |
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
| [`ScanParams`](#scanparams) | struct | Params of `ContentSource.scan`: one page of one account's scan pass. The host injects that account's sealed credentials so the plugin does not need a private credential store under `pluginDataDir`, and drives the pass page by page: the first call of a pass sends an empty `cursor`, every later call echoes the previous page's `nextCursor`. |
| [`MediaSupport`](#mediasupport) | struct | Media a plugin works with, declared in `describe()` and in the manifest's `\[media\]` table. Required for plugins that export the storefront or metadata entrypoint: the host refuses to spawn one whose `workKinds` or `lists` is empty, holds `unspecified`, or is wider than its manifest, and refuses any record whose kinds fall outside the declaration. |
| [`Identifier`](#identifier) | struct | One external identifier a storefront or metadata provider reports. |
| [`Credit`](#credit) | struct | One credited contributor. Credits are listed in billing order. |
| [`SeriesEntry`](#seriesentry) | struct | A work's place in one series. |
| [`ScanBook`](#scanbook) | struct | One library title returned by `ContentSource.scan`. The host forces `source` to the plugin id and, until the library stores works, expressions and editions, maps the record onto one flat library row. |
| [`ScanPage`](#scanpage) | struct | One page of an account's scan pass, the result of `ContentSource.scan`. A pass is the run of pages from an empty `cursor` to the page whose `nextCursor` is empty; `syncToken` and `complete` are read from that last page only. A guest that does not page answers the first call with every title, an empty `nextCursor` and `complete = true`. |
| [`FetchOptions`](#fetchoptions) | struct | Fetch-relevant acquire knobs the host forwards so external load matches in-process. Packaging / naming knobs stay host-side. |
| [`FetchTitleParams`](#fetchtitleparams) | struct | Params of `ContentSource.fetchTitle`. The plugin writes media under `cacheDir` and returns plain (DRM-free) paths. The host injects credentials; guests must not open `library.db` or `master.key`. |
| [`FetchedFile`](#fetchedfile) | struct | One file `fetchTitle` wrote under the cache directory. |
| [`ChapterMarker`](#chaptermarker) | struct | One chapter marker of a fetched title. |
| [`PlainFetch`](#plainfetch) | struct | Plain (DRM-free) fetch result. Sources always return decrypted media; DRM guests decrypt before responding. |
| [`SourceAccounts`](#sourceaccounts) | struct | Success payload of `ContentSource.listAccounts`. |
| [`SearchCatalogParams`](#searchcatalogparams) | struct | Params of `ContentSource.searchCatalog`. |
| [`ExpandCandidatesParams`](#expandcandidatesparams) | struct | Params of `ContentSource.expandCandidates`. Seed fields identify a known title; the guest returns related catalog hits. |
| [`PurchaseHintParams`](#purchasehintparams) | struct | Params of `ContentSource.purchaseHint`. At least one identity field (`productId` / `asin` / `isbn` / title+authors) should be set; guests may return `invalid_params` when none are usable. |
| [`ListDealsParams`](#listdealsparams) | struct | Params of `ContentSource.listDeals`. |
| [`CatalogDetailParams`](#catalogdetailparams) | struct | Params of `ContentSource.catalogDetail`. |
| [`RevokeAccountParams`](#revokeaccountparams) | struct | Params of `ContentSource.revokeAccount`. The host sends the credentials it is about to delete so the guest can sign the device out at the store. |
| [`CatalogHit`](#cataloghit) | struct | One catalog / candidate hit returned by `searchCatalog`, `expandCandidates`, `listDeals`, and `catalogDetail`. |
| [`CatalogHits`](#cataloghits) | struct | Success payload of the catalog list methods. |
| [`CatalogDetail`](#catalogdetail) | struct | Success payload of `ContentSource.catalogDetail`. |
| [`PurchaseHint`](#purchasehint) | struct | Purchase hint for SPA / CLI purchase deep-links. |
| [`PurchaseHintResult`](#purchasehintresult) | struct | Success payload of `ContentSource.purchaseHint`. |
| [`PricedRecord`](#pricedrecord) | struct | One priced listing returned by a title search. |
| [`PurchaseHintsParams`](#purchasehintsparams) | struct | Params of `ContentSource.purchaseHints`: the titles to price together. |
| [`PurchaseHintsResult`](#purchasehintsresult) | struct | Success payload of `ContentSource.purchaseHints`. |
| [`QualityOffer`](#qualityoffer) | struct | The stream one account would download for a title today. |
| [`QualityOffersParams`](#qualityoffersparams) | struct | Params of `ContentSource.qualityOffers`: the titles one account holds. |
| [`TitleQualityOffer`](#titlequalityoffer) | struct | One title's answer in `QualityOffersResult`. |
| [`QualityOffersResult`](#qualityoffersresult) | struct | Success payload of `ContentSource.qualityOffers`. |
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
| [`ScheduledController`](#scheduledcontroller) | struct | One fire of a `\[triggers\] crons` schedule (`scheduled(controller)`). |
| [`MetadataLookupParams`](#metadatalookupparams) | struct | Params of `MetadataProvider.lookup`. |
| [`MetadataSearchParams`](#metadatasearchparams) | struct | Params of `MetadataProvider.search`. |
| [`MetadataChaptersParams`](#metadatachaptersparams) | struct | Params of `MetadataProvider.chapters`. |
| [`MetadataRecord`](#metadatarecord) | struct | One edition as a metadata provider describes it: the same structured fields as `CatalogHit`, without store pricing. |
| [`MetadataRecords`](#metadatarecords) | struct | Success payload of `MetadataProvider.lookup` / `search`. |
| [`MetadataChapter`](#metadatachapter) | struct | One chapter in a provider's chapter layout. Chapters are listed in pre-order: a chapter with `depth` n + 1 is nested under the nearest earlier chapter with `depth` n. |
| [`MetadataChapters`](#metadatachapters) | struct | Success payload of `MetadataProvider.chapters`. |
| [`CoverFetchParams`](#coverfetchparams) | struct | Params of `CoverFetcher.fetchCovers`. |
| [`CoverCondition`](#covercondition) | struct | Validators the host already holds for one URL, sent as conditional request headers so an unchanged image costs a `304` instead of its body. |
| [`CoverImage`](#coverimage) | struct | What happened to one requested URL. |
| [`CoverImages`](#coverimages) | struct | Success payload of `CoverFetcher.fetchCovers`. |
| [`MetadataFileRecord`](#metadatafilerecord) | struct | What one metadata file says about the edition it sits beside. A field the file does not state is left out of `present` and means "unknown", never "cleared"; a field in `present` with an empty value means the file states that it is empty. The host checks every value against the `maxMetadataFile*` caps and refuses a record that states a field its dialect does not carry. |
| [`MetadataFileParseParams`](#metadatafileparseparams) | struct | Params of `MetadataFiles.parse`. |
| [`MetadataFileParsed`](#metadatafileparsed) | struct | Success payload of `MetadataFiles.parse`. |
| [`MetadataAudioFile`](#metadataaudiofile) | struct | One audio file of the edition a sidecar describes, for formats that name their audio (a CUE sheet's `FILE` lines). |
| [`MetadataFileRenderParams`](#metadatafilerenderparams) | struct | Params of `MetadataFiles.render`. |
| [`MetadataFileRendered`](#metadatafilerendered) | struct | Success payload of `MetadataFiles.render`. |
| [`NotificationField`](#notificationfield) | struct | One labelled value shown with a notification. |
| [`NotificationMessage`](#notificationmessage) | struct | A message the host rendered for one notification. Every link is built from the library's public origin, never from a request. |
| [`NotifyDestination`](#notifydestination) | struct | Where a `Notifier.send` goes. |
| [`NotifySendParams`](#notifysendparams) | struct | Params of `Notifier.send`. |
| [`NotifySent`](#notifysent) | struct | Success payload of `Notifier.send`. |
| [`NotifyValidateParams`](#notifyvalidateparams) | struct | Params of `Notifier.validateDestination`. |
| [`NotifyValidated`](#notifyvalidated) | struct | Success payload of `Notifier.validateDestination`. |
| [`NotifyRaiseParams`](#notifyraiseparams) | struct | Params of `NotificationRaiser.raise`. |
| [`NotifyRaised`](#notifyraised) | struct | Success payload of `NotificationRaiser.raise`. |
| [`AiModel`](#aimodel) | struct | One model an `ai` plugin serves. |
| [`AiModelList`](#aimodellist) | struct | Success payload of `AiModels.models`. |
| [`AiEmbedText`](#aiembedtext) | struct | One text to embed. |
| [`AiTextEmbeddingsInput`](#aitextembeddingsinput) | struct | Input of the `textEmbeddings` task. |
| [`AiTextEmbeddingsOutput`](#aitextembeddingsoutput) | struct | Output of the `textEmbeddings` task. |
| [`AiInput`](#aiinput) | struct | Input to one model run: one field per `AiTask`, and exactly one is set (the others are null). A new task appends a field. |
| [`AiOutput`](#aioutput) | struct | Output of one model run: one field per `AiTask`, and exactly one is set (the others are null). A new task appends a field. |
| [`AiRunParams`](#airunparams) | struct | Params of `AiModels.run` and `AiRunner.run`. |
| [`AiRunOk`](#airunok) | struct | Success payload of a model run. |
| [`SocketHeader`](#socketheader) | struct | One request header on `Sockets.connect`. |
| [`SocketAutoResponse`](#socketautoresponse) | struct | A heartbeat the broker answers itself: when `request` arrives byte for byte, it sends `response` and does not wake the guest. |
| [`SocketConnectRequest`](#socketconnectrequest) | struct | Arguments of `Sockets.connect`. |
| [`SocketInfo`](#socketinfo) | struct | One open host-brokered socket. |
| [`SocketList`](#socketlist) | struct | Success payload of `Sockets.list`. |
| [`SocketMessageOk`](#socketmessageok) | struct | What a guest asks of the host after handling one pushed message. |
| [`SocketMessage`](#socketmessage) | struct | Argument of `SocketHandler.message`. |
| [`SocketClosed`](#socketclosed) | struct | Argument of `SocketHandler.closed`. |
| [`OriginHeader`](#originheader) | struct | One header field on an `Origin.fetch` request or response. |
| [`OriginRequest`](#originrequest) | struct | The head of one public request a relay `ingress` plugin carries (`Origin.fetch`). Its scheme is `https` and its authority is the library's public origin; neither is the plugin's to choose. |
| [`OriginRefusal`](#originrefusal) | struct | The library's answer to an `Origin.connect` it did not upgrade: an ordinary response (`401`, `403`, `404`, ...) for the plugin to hand back to the client. |
| [`IngressStarted`](#ingressstarted) | struct | Success payload of `Ingress.start`. |
| [`WebhookUrl`](#webhookurl) | struct | Success payload of `Webhooks.url`. |
| [`WebhookPullRequest`](#webhookpullrequest) | struct | Arguments of `Webhooks.pull`. |
| [`WebhookHeader`](#webhookheader) | struct | One request header kept with a delivery. |
| [`WebhookDelivery`](#webhookdelivery) | struct | One stored request, leased to the guest. |
| [`WebhookPullPage`](#webhookpullpage) | struct | Success payload of `Webhooks.pull`. |
| [`IngressHealth`](#ingresshealth) | struct | Success payload of `Ingress.health`. |
| [`WebhookAckRequest`](#webhookackrequest) | struct | Arguments of `Webhooks.ack`. |
| [`WebhookDelivered`](#webhookdelivered) | struct | Argument of `WebhookHandler.delivered`. |
| [`ProvisionGrant`](#provisiongrant) | struct | A short-lived credential for one guided setup run. The host never stores or logs it; the plugin uses it only during the call. |
| [`ProvisionRequest`](#provisionrequest) | struct | Arguments of `Provisioning.provision`. |
| [`ProvisionedSecret`](#provisionedsecret) | struct | One credential a guided setup returns for the host to seal. |
| [`Provisioned`](#provisioned) | struct | Success payload of `Provisioning.provision`. |
| [`DeprovisionRequest`](#deprovisionrequest) | struct | Arguments of `Provisioning.deprovision`. |
| [`EventPollParams`](#eventpollparams) | struct | Params of `RemoteLibrary.pollEvents`. |
| [`EventAckParams`](#eventackparams) | struct | Params of `RemoteLibrary.ackEvents`. |
| [`Entrypoint`](#entrypoint) | enum | Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint analogue). Each value is a capability the host calls over RPC; triggers on the default entrypoint (`event`, `job`) are declared separately. |
| [`PortalAuthMode`](#portalauthmode) | enum | Portal Accounts connect mode for storefronts. |
| [`CliArgKind`](#cliargkind) | enum | Value kind for a `CliArgSpec`. |
| [`WorkKind`](#workkind) | enum | What family of creative work a record describes (FRBR work). Every record names one: `unspecified` is refused by the host, so a guest that forgets the field fails instead of silently describing a book. Whether a podcast work is a show or an episode is structural, not a kind: an episode names its show through `parentProductId`, a show names no parent. |
| [`ExpressionKind`](#expressionkind) | enum | How a work is realised (FRBR expression): the kind of content you get, independent of the carrier. A translation is not a kind: the expression's language and its translator credits say it. `unspecified` is refused. |
| [`MediaKind`](#mediakind) | enum | Carrier of an edition (FRBR manifestation): what form you receive. Whether the host acquires an edition depends on the storefront's `MediaSupport.fetches` and what the host supports, not on this value alone. `unspecified` is refused. |
| [`IdentifierScheme`](#identifierscheme) | enum | Scheme of an external identifier. The scheme fixes whether the id names a work or an edition, and whether it is strong (unique) or weak (shared). |
| [`CreditRole`](#creditrole) | enum | A contributor's responsibility for a work, expression or edition. |
| [`CreditLevel`](#creditlevel) | enum | Which level a credit attaches to. Ordinal 0 is the role's usual level, so a guest sets this only when a credit departs from it. |
| [`FetchedFileRole`](#fetchedfilerole) | enum | What one fetched file is to the edition it belongs to. |
| [`ChapterKind`](#chapterkind) | enum | What a storefront's chapter markers mark (`PlainFetch.chapterKind`). |
| [`CatalogSort`](#catalogsort) | enum | Catalog search ordering. |
| [`CatalogField`](#catalogfield) | enum | Catalog search facet restricting which field the query matches. |
| [`PriceLookup`](#pricelookup) | enum | How a purchase-hint query names the title to price. |
| [`Abridgement`](#abridgement) | enum | Whether an edition is abridged. |
| [`Explicitness`](#explicitness) | enum | Whether an edition is marked as explicit (adult) content. A patron scope can hide explicit editions; `unknown` and `clean` editions stay visible. |
| [`OfferedCodec`](#offeredcodec) | enum | Audio codec of a stream a store offers (`QualityOffer.codec`). Spelled as the host's probed codecs are, so an offer compares with a held file. |
| [`OfferedAacProfile`](#offeredaacprofile) | enum | AAC profile of an offered AAC stream (`QualityOffer.aacProfile`). |
| [`InvocationPurpose`](#invocationpurpose) | enum | Why the host opens a plugin's entrypoints (`Invocation.purpose`). |
| [`RemoteMedia`](#remotemedia) | enum | The kind of public media `CoverFetcher.fetchCovers` downloads. |
| [`MetadataField`](#metadatafield) | enum | A field of a `MetadataFileRecord`. Dialects declare in the manifest which fields their format carries; a parsed record lists the ones its file states, and a render request lists the ones to write. |
| [`NotificationSeverity`](#notificationseverity) | enum | How urgent a notification is. Routing rules carry a minimum severity. |
| [`NotificationAudience`](#notificationaudience) | enum | Who a notification type is for. The account audience is reserved to the host; a plugin may declare only operational and patron types. |
| [`NotificationAlertAction`](#notificationalertaction) | enum | Lifecycle step of an alert, for services such as PagerDuty that open and close incidents. |
| [`NotifyDestinationKind`](#notifydestinationkind) | enum | Kind of destination a `Notifier` sends to. |
| [`AiTask`](#aitask) | enum | What a model does. A model has exactly one task, and the task fixes the shape of `AiRunParams.input` and of the reply. |
| [`AiTextRole`](#aitextrole) | enum | What a text is to an embedding model. Some models embed a search query differently from the documents it is matched against. |
| [`ProvisionGrantKind`](#provisiongrantkind) | enum | What kind of grant the operator gave a guided setup (`ProvisionGrant`). |
| [`QueueResult`](#queueresult) | enum | Per-message outcome inside a `QueueConsumer.queue` batch reply. |
| [`SocketFrame`](#socketframe) | enum | One WebSocket message, or one chunk of a `tls://` stream (always `binary`). |
| [`OriginConnected`](#originconnected) | enum | Success payload of `Origin.connect`. |
| [`RoundTripFailure`](#roundtripfailure) | enum | Why `roundtrip_any_message` could not show `read(write(v)) == v`. |
| [`write_event_consumer_spec`](#write_event_consumer_spec) | function | Encode a `EventConsumerSpec` onto a Cap'n Proto builder. |
| [`read_event_consumer_spec`](#read_event_consumer_spec) | function | Decode a `EventConsumerSpec` from a Cap'n Proto reader. |
| [`write_plugin_capabilities`](#write_plugin_capabilities) | function | Encode a `PluginCapabilities` onto a Cap'n Proto builder. |
| [`read_plugin_capabilities`](#read_plugin_capabilities) | function | Decode a `PluginCapabilities` from a Cap'n Proto reader. |
| [`write_catalog_capabilities`](#write_catalog_capabilities) | function | Encode a `CatalogCapabilities` onto a Cap'n Proto builder. |
| [`read_catalog_capabilities`](#read_catalog_capabilities) | function | Decode a `CatalogCapabilities` from a Cap'n Proto reader. |
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
| [`write_media_support`](#write_media_support) | function | Encode a `MediaSupport` onto a Cap'n Proto builder. |
| [`read_media_support`](#read_media_support) | function | Decode a `MediaSupport` from a Cap'n Proto reader. |
| [`write_identifier`](#write_identifier) | function | Encode a `Identifier` onto a Cap'n Proto builder. |
| [`read_identifier`](#read_identifier) | function | Decode a `Identifier` from a Cap'n Proto reader. |
| [`write_credit`](#write_credit) | function | Encode a `Credit` onto a Cap'n Proto builder. |
| [`read_credit`](#read_credit) | function | Decode a `Credit` from a Cap'n Proto reader. |
| [`write_series_entry`](#write_series_entry) | function | Encode a `SeriesEntry` onto a Cap'n Proto builder. |
| [`read_series_entry`](#read_series_entry) | function | Decode a `SeriesEntry` from a Cap'n Proto reader. |
| [`write_scan_book`](#write_scan_book) | function | Encode a `ScanBook` onto a Cap'n Proto builder. |
| [`read_scan_book`](#read_scan_book) | function | Decode a `ScanBook` from a Cap'n Proto reader. |
| [`write_scan_page`](#write_scan_page) | function | Encode a `ScanPage` onto a Cap'n Proto builder. |
| [`read_scan_page`](#read_scan_page) | function | Decode a `ScanPage` from a Cap'n Proto reader. |
| [`write_scan_reply`](#write_scan_reply) | function | Encode a `ScanReply` union from a typed result. |
| [`read_scan_reply`](#read_scan_reply) | function | Decode a `ScanReply` union into a typed result. |
| [`write_fetch_options`](#write_fetch_options) | function | Encode a `FetchOptions` onto a Cap'n Proto builder. |
| [`read_fetch_options`](#read_fetch_options) | function | Decode a `FetchOptions` from a Cap'n Proto reader. |
| [`write_fetch_title_params`](#write_fetch_title_params) | function | Encode a `FetchTitleParams` onto a Cap'n Proto builder. |
| [`read_fetch_title_params`](#read_fetch_title_params) | function | Decode a `FetchTitleParams` from a Cap'n Proto reader. |
| [`write_fetched_file`](#write_fetched_file) | function | Encode a `FetchedFile` onto a Cap'n Proto builder. |
| [`read_fetched_file`](#read_fetched_file) | function | Decode a `FetchedFile` from a Cap'n Proto reader. |
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
| [`write_revoke_account_params`](#write_revoke_account_params) | function | Encode a `RevokeAccountParams` onto a Cap'n Proto builder. |
| [`read_revoke_account_params`](#read_revoke_account_params) | function | Decode a `RevokeAccountParams` from a Cap'n Proto reader. |
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
| [`write_priced_record`](#write_priced_record) | function | Encode a `PricedRecord` onto a Cap'n Proto builder. |
| [`read_priced_record`](#read_priced_record) | function | Decode a `PricedRecord` from a Cap'n Proto reader. |
| [`write_purchase_hint_reply`](#write_purchase_hint_reply) | function | Encode a `PurchaseHintReply` union from a typed result. |
| [`read_purchase_hint_reply`](#read_purchase_hint_reply) | function | Decode a `PurchaseHintReply` union into a typed result. |
| [`write_purchase_hints_params`](#write_purchase_hints_params) | function | Encode a `PurchaseHintsParams` onto a Cap'n Proto builder. |
| [`read_purchase_hints_params`](#read_purchase_hints_params) | function | Decode a `PurchaseHintsParams` from a Cap'n Proto reader. |
| [`write_purchase_hints_result`](#write_purchase_hints_result) | function | Encode a `PurchaseHintsResult` onto a Cap'n Proto builder. |
| [`read_purchase_hints_result`](#read_purchase_hints_result) | function | Decode a `PurchaseHintsResult` from a Cap'n Proto reader. |
| [`write_purchase_hints_reply`](#write_purchase_hints_reply) | function | Encode a `PurchaseHintsReply` union from a typed result. |
| [`read_purchase_hints_reply`](#read_purchase_hints_reply) | function | Decode a `PurchaseHintsReply` union into a typed result. |
| [`write_quality_offer`](#write_quality_offer) | function | Encode a `QualityOffer` onto a Cap'n Proto builder. |
| [`read_quality_offer`](#read_quality_offer) | function | Decode a `QualityOffer` from a Cap'n Proto reader. |
| [`write_quality_offers_params`](#write_quality_offers_params) | function | Encode a `QualityOffersParams` onto a Cap'n Proto builder. |
| [`read_quality_offers_params`](#read_quality_offers_params) | function | Decode a `QualityOffersParams` from a Cap'n Proto reader. |
| [`write_title_quality_offer`](#write_title_quality_offer) | function | Encode a `TitleQualityOffer` onto a Cap'n Proto builder. |
| [`read_title_quality_offer`](#read_title_quality_offer) | function | Decode a `TitleQualityOffer` from a Cap'n Proto reader. |
| [`write_quality_offers_result`](#write_quality_offers_result) | function | Encode a `QualityOffersResult` onto a Cap'n Proto builder. |
| [`read_quality_offers_result`](#read_quality_offers_result) | function | Decode a `QualityOffersResult` from a Cap'n Proto reader. |
| [`write_quality_offers_reply`](#write_quality_offers_reply) | function | Encode a `QualityOffersReply` union from a typed result. |
| [`read_quality_offers_reply`](#read_quality_offers_reply) | function | Decode a `QualityOffersReply` union into a typed result. |
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
| [`write_scheduled_controller`](#write_scheduled_controller) | function | Encode a `ScheduledController` onto a Cap'n Proto builder. |
| [`read_scheduled_controller`](#read_scheduled_controller) | function | Decode a `ScheduledController` from a Cap'n Proto reader. |
| [`write_metadata_lookup_params`](#write_metadata_lookup_params) | function | Encode a `MetadataLookupParams` onto a Cap'n Proto builder. |
| [`read_metadata_lookup_params`](#read_metadata_lookup_params) | function | Decode a `MetadataLookupParams` from a Cap'n Proto reader. |
| [`write_metadata_search_params`](#write_metadata_search_params) | function | Encode a `MetadataSearchParams` onto a Cap'n Proto builder. |
| [`read_metadata_search_params`](#read_metadata_search_params) | function | Decode a `MetadataSearchParams` from a Cap'n Proto reader. |
| [`write_metadata_chapters_params`](#write_metadata_chapters_params) | function | Encode a `MetadataChaptersParams` onto a Cap'n Proto builder. |
| [`read_metadata_chapters_params`](#read_metadata_chapters_params) | function | Decode a `MetadataChaptersParams` from a Cap'n Proto reader. |
| [`write_metadata_record`](#write_metadata_record) | function | Encode a `MetadataRecord` onto a Cap'n Proto builder. |
| [`read_metadata_record`](#read_metadata_record) | function | Decode a `MetadataRecord` from a Cap'n Proto reader. |
| [`write_metadata_records`](#write_metadata_records) | function | Encode a `MetadataRecords` onto a Cap'n Proto builder. |
| [`read_metadata_records`](#read_metadata_records) | function | Decode a `MetadataRecords` from a Cap'n Proto reader. |
| [`write_metadata_records_reply`](#write_metadata_records_reply) | function | Encode a `MetadataRecordsReply` union from a typed result. |
| [`read_metadata_records_reply`](#read_metadata_records_reply) | function | Decode a `MetadataRecordsReply` union into a typed result. |
| [`write_metadata_chapter`](#write_metadata_chapter) | function | Encode a `MetadataChapter` onto a Cap'n Proto builder. |
| [`read_metadata_chapter`](#read_metadata_chapter) | function | Decode a `MetadataChapter` from a Cap'n Proto reader. |
| [`write_metadata_chapters`](#write_metadata_chapters) | function | Encode a `MetadataChapters` onto a Cap'n Proto builder. |
| [`read_metadata_chapters`](#read_metadata_chapters) | function | Decode a `MetadataChapters` from a Cap'n Proto reader. |
| [`write_metadata_chapters_reply`](#write_metadata_chapters_reply) | function | Encode a `MetadataChaptersReply` union from a typed result. |
| [`read_metadata_chapters_reply`](#read_metadata_chapters_reply) | function | Decode a `MetadataChaptersReply` union into a typed result. |
| [`write_cover_fetch_params`](#write_cover_fetch_params) | function | Encode a `CoverFetchParams` onto a Cap'n Proto builder. |
| [`read_cover_fetch_params`](#read_cover_fetch_params) | function | Decode a `CoverFetchParams` from a Cap'n Proto reader. |
| [`write_cover_condition`](#write_cover_condition) | function | Encode a `CoverCondition` onto a Cap'n Proto builder. |
| [`read_cover_condition`](#read_cover_condition) | function | Decode a `CoverCondition` from a Cap'n Proto reader. |
| [`write_cover_image`](#write_cover_image) | function | Encode a `CoverImage` onto a Cap'n Proto builder. |
| [`read_cover_image`](#read_cover_image) | function | Decode a `CoverImage` from a Cap'n Proto reader. |
| [`write_cover_images`](#write_cover_images) | function | Encode a `CoverImages` onto a Cap'n Proto builder. |
| [`read_cover_images`](#read_cover_images) | function | Decode a `CoverImages` from a Cap'n Proto reader. |
| [`write_cover_images_reply`](#write_cover_images_reply) | function | Encode a `CoverImagesReply` union from a typed result. |
| [`read_cover_images_reply`](#read_cover_images_reply) | function | Decode a `CoverImagesReply` union into a typed result. |
| [`write_metadata_file_record`](#write_metadata_file_record) | function | Encode a `MetadataFileRecord` onto a Cap'n Proto builder. |
| [`read_metadata_file_record`](#read_metadata_file_record) | function | Decode a `MetadataFileRecord` from a Cap'n Proto reader. |
| [`write_metadata_file_parse_params`](#write_metadata_file_parse_params) | function | Encode a `MetadataFileParseParams` onto a Cap'n Proto builder. |
| [`read_metadata_file_parse_params`](#read_metadata_file_parse_params) | function | Decode a `MetadataFileParseParams` from a Cap'n Proto reader. |
| [`write_metadata_file_parsed`](#write_metadata_file_parsed) | function | Encode a `MetadataFileParsed` onto a Cap'n Proto builder. |
| [`read_metadata_file_parsed`](#read_metadata_file_parsed) | function | Decode a `MetadataFileParsed` from a Cap'n Proto reader. |
| [`write_metadata_file_parse_reply`](#write_metadata_file_parse_reply) | function | Encode a `MetadataFileParseReply` union from a typed result. |
| [`read_metadata_file_parse_reply`](#read_metadata_file_parse_reply) | function | Decode a `MetadataFileParseReply` union into a typed result. |
| [`write_metadata_audio_file`](#write_metadata_audio_file) | function | Encode a `MetadataAudioFile` onto a Cap'n Proto builder. |
| [`read_metadata_audio_file`](#read_metadata_audio_file) | function | Decode a `MetadataAudioFile` from a Cap'n Proto reader. |
| [`write_metadata_file_render_params`](#write_metadata_file_render_params) | function | Encode a `MetadataFileRenderParams` onto a Cap'n Proto builder. |
| [`read_metadata_file_render_params`](#read_metadata_file_render_params) | function | Decode a `MetadataFileRenderParams` from a Cap'n Proto reader. |
| [`write_metadata_file_rendered`](#write_metadata_file_rendered) | function | Encode a `MetadataFileRendered` onto a Cap'n Proto builder. |
| [`read_metadata_file_rendered`](#read_metadata_file_rendered) | function | Decode a `MetadataFileRendered` from a Cap'n Proto reader. |
| [`write_metadata_file_render_reply`](#write_metadata_file_render_reply) | function | Encode a `MetadataFileRenderReply` union from a typed result. |
| [`read_metadata_file_render_reply`](#read_metadata_file_render_reply) | function | Decode a `MetadataFileRenderReply` union into a typed result. |
| [`write_notification_field`](#write_notification_field) | function | Encode a `NotificationField` onto a Cap'n Proto builder. |
| [`read_notification_field`](#read_notification_field) | function | Decode a `NotificationField` from a Cap'n Proto reader. |
| [`write_notification_message`](#write_notification_message) | function | Encode a `NotificationMessage` onto a Cap'n Proto builder. |
| [`read_notification_message`](#read_notification_message) | function | Decode a `NotificationMessage` from a Cap'n Proto reader. |
| [`write_notify_destination`](#write_notify_destination) | function | Encode a `NotifyDestination` onto a Cap'n Proto builder. |
| [`read_notify_destination`](#read_notify_destination) | function | Decode a `NotifyDestination` from a Cap'n Proto reader. |
| [`write_notify_send_params`](#write_notify_send_params) | function | Encode a `NotifySendParams` onto a Cap'n Proto builder. |
| [`read_notify_send_params`](#read_notify_send_params) | function | Decode a `NotifySendParams` from a Cap'n Proto reader. |
| [`write_notify_sent`](#write_notify_sent) | function | Encode a `NotifySent` onto a Cap'n Proto builder. |
| [`read_notify_sent`](#read_notify_sent) | function | Decode a `NotifySent` from a Cap'n Proto reader. |
| [`write_notify_send_reply`](#write_notify_send_reply) | function | Encode a `NotifySendReply` union from a typed result. |
| [`read_notify_send_reply`](#read_notify_send_reply) | function | Decode a `NotifySendReply` union into a typed result. |
| [`write_notify_validate_params`](#write_notify_validate_params) | function | Encode a `NotifyValidateParams` onto a Cap'n Proto builder. |
| [`read_notify_validate_params`](#read_notify_validate_params) | function | Decode a `NotifyValidateParams` from a Cap'n Proto reader. |
| [`write_notify_validated`](#write_notify_validated) | function | Encode a `NotifyValidated` onto a Cap'n Proto builder. |
| [`read_notify_validated`](#read_notify_validated) | function | Decode a `NotifyValidated` from a Cap'n Proto reader. |
| [`write_notify_validate_reply`](#write_notify_validate_reply) | function | Encode a `NotifyValidateReply` union from a typed result. |
| [`read_notify_validate_reply`](#read_notify_validate_reply) | function | Decode a `NotifyValidateReply` union into a typed result. |
| [`write_notify_raise_params`](#write_notify_raise_params) | function | Encode a `NotifyRaiseParams` onto a Cap'n Proto builder. |
| [`read_notify_raise_params`](#read_notify_raise_params) | function | Decode a `NotifyRaiseParams` from a Cap'n Proto reader. |
| [`write_notify_raised`](#write_notify_raised) | function | Encode a `NotifyRaised` onto a Cap'n Proto builder. |
| [`read_notify_raised`](#read_notify_raised) | function | Decode a `NotifyRaised` from a Cap'n Proto reader. |
| [`write_notify_raise_reply`](#write_notify_raise_reply) | function | Encode a `NotifyRaiseReply` union from a typed result. |
| [`read_notify_raise_reply`](#read_notify_raise_reply) | function | Decode a `NotifyRaiseReply` union into a typed result. |
| [`write_ai_model`](#write_ai_model) | function | Encode a `AiModel` onto a Cap'n Proto builder. |
| [`read_ai_model`](#read_ai_model) | function | Decode a `AiModel` from a Cap'n Proto reader. |
| [`write_ai_model_list`](#write_ai_model_list) | function | Encode a `AiModelList` onto a Cap'n Proto builder. |
| [`read_ai_model_list`](#read_ai_model_list) | function | Decode a `AiModelList` from a Cap'n Proto reader. |
| [`write_ai_models_reply`](#write_ai_models_reply) | function | Encode a `AiModelsReply` union from a typed result. |
| [`read_ai_models_reply`](#read_ai_models_reply) | function | Decode a `AiModelsReply` union into a typed result. |
| [`write_ai_embed_text`](#write_ai_embed_text) | function | Encode a `AiEmbedText` onto a Cap'n Proto builder. |
| [`read_ai_embed_text`](#read_ai_embed_text) | function | Decode a `AiEmbedText` from a Cap'n Proto reader. |
| [`write_ai_text_embeddings_input`](#write_ai_text_embeddings_input) | function | Encode a `AiTextEmbeddingsInput` onto a Cap'n Proto builder. |
| [`read_ai_text_embeddings_input`](#read_ai_text_embeddings_input) | function | Decode a `AiTextEmbeddingsInput` from a Cap'n Proto reader. |
| [`write_ai_text_embeddings_output`](#write_ai_text_embeddings_output) | function | Encode a `AiTextEmbeddingsOutput` onto a Cap'n Proto builder. |
| [`read_ai_text_embeddings_output`](#read_ai_text_embeddings_output) | function | Decode a `AiTextEmbeddingsOutput` from a Cap'n Proto reader. |
| [`write_ai_input`](#write_ai_input) | function | Encode a `AiInput` onto a Cap'n Proto builder. |
| [`read_ai_input`](#read_ai_input) | function | Decode a `AiInput` from a Cap'n Proto reader. |
| [`write_ai_output`](#write_ai_output) | function | Encode a `AiOutput` onto a Cap'n Proto builder. |
| [`read_ai_output`](#read_ai_output) | function | Decode a `AiOutput` from a Cap'n Proto reader. |
| [`write_ai_run_params`](#write_ai_run_params) | function | Encode a `AiRunParams` onto a Cap'n Proto builder. |
| [`read_ai_run_params`](#read_ai_run_params) | function | Decode a `AiRunParams` from a Cap'n Proto reader. |
| [`write_ai_run_ok`](#write_ai_run_ok) | function | Encode a `AiRunOk` onto a Cap'n Proto builder. |
| [`read_ai_run_ok`](#read_ai_run_ok) | function | Decode a `AiRunOk` from a Cap'n Proto reader. |
| [`write_ai_run_reply`](#write_ai_run_reply) | function | Encode a `AiRunReply` union from a typed result. |
| [`read_ai_run_reply`](#read_ai_run_reply) | function | Decode a `AiRunReply` union into a typed result. |
| [`write_socket_frame`](#write_socket_frame) | function | Encode a `SocketFrame` union. |
| [`read_socket_frame`](#read_socket_frame) | function | Decode a `SocketFrame` union. |
| [`write_socket_header`](#write_socket_header) | function | Encode a `SocketHeader` onto a Cap'n Proto builder. |
| [`read_socket_header`](#read_socket_header) | function | Decode a `SocketHeader` from a Cap'n Proto reader. |
| [`write_socket_auto_response`](#write_socket_auto_response) | function | Encode a `SocketAutoResponse` onto a Cap'n Proto builder. |
| [`read_socket_auto_response`](#read_socket_auto_response) | function | Decode a `SocketAutoResponse` from a Cap'n Proto reader. |
| [`write_socket_connect_request`](#write_socket_connect_request) | function | Encode a `SocketConnectRequest` onto a Cap'n Proto builder. |
| [`read_socket_connect_request`](#read_socket_connect_request) | function | Decode a `SocketConnectRequest` from a Cap'n Proto reader. |
| [`write_socket_info`](#write_socket_info) | function | Encode a `SocketInfo` onto a Cap'n Proto builder. |
| [`read_socket_info`](#read_socket_info) | function | Decode a `SocketInfo` from a Cap'n Proto reader. |
| [`write_socket_connect_reply`](#write_socket_connect_reply) | function | Encode a `SocketConnectReply` union from a typed result. |
| [`read_socket_connect_reply`](#read_socket_connect_reply) | function | Decode a `SocketConnectReply` union into a typed result. |
| [`write_socket_list`](#write_socket_list) | function | Encode a `SocketList` onto a Cap'n Proto builder. |
| [`read_socket_list`](#read_socket_list) | function | Decode a `SocketList` from a Cap'n Proto reader. |
| [`write_socket_list_reply`](#write_socket_list_reply) | function | Encode a `SocketListReply` union from a typed result. |
| [`read_socket_list_reply`](#read_socket_list_reply) | function | Decode a `SocketListReply` union into a typed result. |
| [`write_socket_message_reply`](#write_socket_message_reply) | function | Encode a `SocketMessageReply` union from a typed result. |
| [`read_socket_message_reply`](#read_socket_message_reply) | function | Decode a `SocketMessageReply` union into a typed result. |
| [`write_socket_message_ok`](#write_socket_message_ok) | function | Encode a `SocketMessageOk` onto a Cap'n Proto builder. |
| [`read_socket_message_ok`](#read_socket_message_ok) | function | Decode a `SocketMessageOk` from a Cap'n Proto reader. |
| [`write_socket_message`](#write_socket_message) | function | Encode a `SocketMessage` onto a Cap'n Proto builder. |
| [`read_socket_message`](#read_socket_message) | function | Decode a `SocketMessage` from a Cap'n Proto reader. |
| [`write_socket_closed`](#write_socket_closed) | function | Encode a `SocketClosed` onto a Cap'n Proto builder. |
| [`read_socket_closed`](#read_socket_closed) | function | Decode a `SocketClosed` from a Cap'n Proto reader. |
| [`write_origin_header`](#write_origin_header) | function | Encode a `OriginHeader` onto a Cap'n Proto builder. |
| [`read_origin_header`](#read_origin_header) | function | Decode a `OriginHeader` from a Cap'n Proto reader. |
| [`write_origin_request`](#write_origin_request) | function | Encode a `OriginRequest` onto a Cap'n Proto builder. |
| [`read_origin_request`](#read_origin_request) | function | Decode a `OriginRequest` from a Cap'n Proto reader. |
| [`write_origin_refusal`](#write_origin_refusal) | function | Encode a `OriginRefusal` onto a Cap'n Proto builder. |
| [`read_origin_refusal`](#read_origin_refusal) | function | Decode a `OriginRefusal` from a Cap'n Proto reader. |
| [`write_origin_connected`](#write_origin_connected) | function | Encode a `OriginConnected` union. |
| [`read_origin_connected`](#read_origin_connected) | function | Decode a `OriginConnected` union. |
| [`write_origin_connect_reply`](#write_origin_connect_reply) | function | Encode a `OriginConnectReply` union from a typed result. |
| [`read_origin_connect_reply`](#read_origin_connect_reply) | function | Decode a `OriginConnectReply` union into a typed result. |
| [`write_ingress_started`](#write_ingress_started) | function | Encode a `IngressStarted` onto a Cap'n Proto builder. |
| [`read_ingress_started`](#read_ingress_started) | function | Decode a `IngressStarted` from a Cap'n Proto reader. |
| [`write_ingress_start_reply`](#write_ingress_start_reply) | function | Encode a `IngressStartReply` union from a typed result. |
| [`read_ingress_start_reply`](#read_ingress_start_reply) | function | Decode a `IngressStartReply` union into a typed result. |
| [`write_webhook_url`](#write_webhook_url) | function | Encode a `WebhookUrl` onto a Cap'n Proto builder. |
| [`read_webhook_url`](#read_webhook_url) | function | Decode a `WebhookUrl` from a Cap'n Proto reader. |
| [`write_webhook_url_reply`](#write_webhook_url_reply) | function | Encode a `WebhookUrlReply` union from a typed result. |
| [`read_webhook_url_reply`](#read_webhook_url_reply) | function | Decode a `WebhookUrlReply` union into a typed result. |
| [`write_webhook_pull_request`](#write_webhook_pull_request) | function | Encode a `WebhookPullRequest` onto a Cap'n Proto builder. |
| [`read_webhook_pull_request`](#read_webhook_pull_request) | function | Decode a `WebhookPullRequest` from a Cap'n Proto reader. |
| [`write_webhook_header`](#write_webhook_header) | function | Encode a `WebhookHeader` onto a Cap'n Proto builder. |
| [`read_webhook_header`](#read_webhook_header) | function | Decode a `WebhookHeader` from a Cap'n Proto reader. |
| [`write_webhook_delivery`](#write_webhook_delivery) | function | Encode a `WebhookDelivery` onto a Cap'n Proto builder. |
| [`read_webhook_delivery`](#read_webhook_delivery) | function | Decode a `WebhookDelivery` from a Cap'n Proto reader. |
| [`write_webhook_pull_page`](#write_webhook_pull_page) | function | Encode a `WebhookPullPage` onto a Cap'n Proto builder. |
| [`read_webhook_pull_page`](#read_webhook_pull_page) | function | Decode a `WebhookPullPage` from a Cap'n Proto reader. |
| [`write_webhook_pull_reply`](#write_webhook_pull_reply) | function | Encode a `WebhookPullReply` union from a typed result. |
| [`read_webhook_pull_reply`](#read_webhook_pull_reply) | function | Decode a `WebhookPullReply` union into a typed result. |
| [`write_ingress_health`](#write_ingress_health) | function | Encode a `IngressHealth` onto a Cap'n Proto builder. |
| [`read_ingress_health`](#read_ingress_health) | function | Decode a `IngressHealth` from a Cap'n Proto reader. |
| [`write_ingress_health_reply`](#write_ingress_health_reply) | function | Encode a `IngressHealthReply` union from a typed result. |
| [`read_ingress_health_reply`](#read_ingress_health_reply) | function | Decode a `IngressHealthReply` union into a typed result. |
| [`write_webhook_ack_request`](#write_webhook_ack_request) | function | Encode a `WebhookAckRequest` onto a Cap'n Proto builder. |
| [`read_webhook_ack_request`](#read_webhook_ack_request) | function | Decode a `WebhookAckRequest` from a Cap'n Proto reader. |
| [`write_webhook_delivered`](#write_webhook_delivered) | function | Encode a `WebhookDelivered` onto a Cap'n Proto builder. |
| [`read_webhook_delivered`](#read_webhook_delivered) | function | Decode a `WebhookDelivered` from a Cap'n Proto reader. |
| [`write_provision_grant`](#write_provision_grant) | function | Encode a `ProvisionGrant` onto a Cap'n Proto builder. |
| [`read_provision_grant`](#read_provision_grant) | function | Decode a `ProvisionGrant` from a Cap'n Proto reader. |
| [`write_provision_request`](#write_provision_request) | function | Encode a `ProvisionRequest` onto a Cap'n Proto builder. |
| [`read_provision_request`](#read_provision_request) | function | Decode a `ProvisionRequest` from a Cap'n Proto reader. |
| [`write_provisioned_secret`](#write_provisioned_secret) | function | Encode a `ProvisionedSecret` onto a Cap'n Proto builder. |
| [`read_provisioned_secret`](#read_provisioned_secret) | function | Decode a `ProvisionedSecret` from a Cap'n Proto reader. |
| [`write_provisioned`](#write_provisioned) | function | Encode a `Provisioned` onto a Cap'n Proto builder. |
| [`read_provisioned`](#read_provisioned) | function | Decode a `Provisioned` from a Cap'n Proto reader. |
| [`write_provision_reply`](#write_provision_reply) | function | Encode a `ProvisionReply` union from a typed result. |
| [`read_provision_reply`](#read_provision_reply) | function | Decode a `ProvisionReply` union into a typed result. |
| [`write_deprovision_request`](#write_deprovision_request) | function | Encode a `DeprovisionRequest` onto a Cap'n Proto builder. |
| [`read_deprovision_request`](#read_deprovision_request) | function | Decode a `DeprovisionRequest` from a Cap'n Proto reader. |
| [`write_event_poll_params`](#write_event_poll_params) | function | Encode a `EventPollParams` onto a Cap'n Proto builder. |
| [`read_event_poll_params`](#read_event_poll_params) | function | Decode a `EventPollParams` from a Cap'n Proto reader. |
| [`write_event_ack_params`](#write_event_ack_params) | function | Encode a `EventAckParams` onto a Cap'n Proto builder. |
| [`read_event_ack_params`](#read_event_ack_params) | function | Decode a `EventAckParams` from a Cap'n Proto reader. |
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
    pub crons: Vec<String>,
    pub sockets: bool,
    pub webhooks: bool,
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
- `crons`: Five-field UTC cron schedules the default entrypoint handles
  (`scheduled(controller)` trigger, `\[triggers\] crons`), normalized to
  single spaces.
- `sockets`: True when the guest declared `\[sockets\]`: it receives the host-brokered
  `Sockets` binding and may export a `SocketHandler` for pushed messages.
- `webhooks`: True when the guest declared `\[\[webhooks\]\]`: it receives the host-served
  `Webhooks` binding and may export a `WebhookHandler` the host wakes when
  it stores deliveries.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogCapabilities

```rust
pub struct CatalogCapabilities {
    pub detail_by_isbn: bool,
    pub price_by_isbn: bool,
    pub price_by_asin: bool,
    pub detail_by_product_id: bool,
    pub quality_offers: bool,
}
```

Catalog behaviours a storefront declares in `describe()`. Catalog search
reads them instead of special-casing plugin ids; every flag defaults to
false, so a guest that says nothing keeps the plain behaviour.

**Fields**

- `detail_by_isbn`: True when `catalogDetail` resolves a bare ISBN (sent in both `productId`
  and `isbn`) to a full bibliographic record. Catalog search then asks this
  store to fill description, narrators, runtime and genres on lean result
  rows from any store that carry an ISBN.
- `price_by_isbn`: True when `purchaseHint` / `purchaseHints` price a query by its exact
  ISBN (`lookup = isbn`): the answer is the store's listing of that ISBN
  or not found, never a title match. The host then prices a card at this
  store by the ISBNs of the card's recording before searching by title.
- `price_by_asin`: True when `purchaseHint` / `purchaseHints` price a query by its exact
  ASIN (`lookup = asin`), as `priceByIsbn` does for ISBNs.
- `detail_by_product_id`: True when `catalogDetail` resolves this store's own `productId` (an id
  its search, expand or deals hits gave) to a full bibliographic record.
  Catalog search then asks this store to fill lean result rows it listed
  itself, and title dialogs ask it for the editions it listed. A store
  that leaves it false is never woken for a detail lookup by product id.
- `quality_offers`: True when `qualityOffers` answers which audio quality an account would
  download for a title. The host's quality-upgrade check asks only stores
  that declare it.

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
    pub account_id: String,
    pub page_size: u32,
    pub import_episodes: bool,
    pub import_plus_titles: bool,
    pub credential: AccountCredential,
    pub cursor: String,
    pub sync_token: String,
}
```

Params of `ContentSource.scan`: one page of one account's scan pass. The
host injects that account's sealed credentials so the plugin does not need a
private credential store under `pluginDataDir`, and drives the pass page by
page: the first call of a pass sends an empty `cursor`, every later call
echoes the previous page's `nextCursor`.

**Fields**

- `plugin_data_dir`: Scoped plugin data directory.
- `account_id`: Account this pass scans; always one of the plugin's connected accounts.
- `page_size`: Storefront page size; the host always sends an explicit value.
- `import_episodes`: When true, import podcast/episode-style rows.
- `import_plus_titles`: When true, import Plus/catalog entitlement titles.
- `credential`: Host-loaded credential blob for `accountId`.
- `cursor`: Opaque position the guest returned as `ScanPage.nextCursor`; empty on the
  first page of a pass. At most `maxScanCursorBytes`.
- `sync_token`: Opaque token the guest returned as `ScanPage.syncToken` at the end of the
  account's last pass; empty asks for a full pass. At most
  `maxScanCursorBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MediaSupport

```rust
pub struct MediaSupport {
    pub work_kinds: Vec<WorkKind>,
    pub lists: Vec<MediaKind>,
    pub fetches: Vec<MediaKind>,
}
```

Media a plugin works with, declared in `describe()` and in the manifest's
`\[media\]` table. Required for plugins that export the storefront or
metadata entrypoint: the host refuses to spawn one whose `workKinds` or
`lists` is empty, holds `unspecified`, or is wider than its manifest, and
refuses any record whose kinds fall outside the declaration.

**Fields**

- `work_kinds`: Work kinds the plugin reports.
- `lists`: Edition carriers the plugin reports in scans, catalog search and
  metadata records.
- `fetches`: Edition carriers `ContentSource.fetchTitle` can deliver; a subset of
  `lists`. Empty for metadata providers and for stores that only list.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Identifier

```rust
pub struct Identifier {
    pub scheme: IdentifierScheme,
    pub value: String,
    pub marketplace: Option<String>,
}
```

One external identifier a storefront or metadata provider reports.

**Fields**

- `scheme`: Scheme the value belongs to; it decides the identifier's level.
- `value`: The identifier as the source printed it; the host trims and normalises.
- `marketplace`: Marketplace the id is scoped to (ASIN); absent for global schemes.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Credit

```rust
pub struct Credit {
    pub name: String,
    pub role: CreditRole,
    pub level: CreditLevel,
}
```

One credited contributor. Credits are listed in billing order.

**Fields**

- `name`: Contributor display name as the source prints it, one person or body.
- `role`: What the contributor did.
- `level`: Level the credit attaches to.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SeriesEntry

```rust
pub struct SeriesEntry {
    pub name: String,
    pub position: Option<String>,
    pub store_series_id: Option<String>,
}
```

A work's place in one series.

**Fields**

- `name`: Series name as the source prints it.
- `position`: Position label as the source prints it (`1`, `2.5`, `1-3`).
  `None` when absent (wire zero value).
- `store_series_id`: The store's own id for the series (an Audible series ASIN) when known.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanBook

```rust
pub struct ScanBook {
    pub account_id: String,
    pub product_id: String,
    pub title: String,
    pub marketplace: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<i64>,
    pub subtitle: Option<String>,
    pub work_kind: WorkKind,
    pub expression_kind: ExpressionKind,
    pub media_kind: MediaKind,
    pub identifiers: Vec<Identifier>,
    pub credits: Vec<Credit>,
    pub series_entries: Vec<SeriesEntry>,
    pub parent_product_id: Option<String>,
    pub language: Option<String>,
    pub abridgement: Abridgement,
    pub published_at: Option<String>,
    pub purchased_at: Option<String>,
    pub categories: Option<String>,
    pub explicitness: Explicitness,
}
```

One library title returned by `ContentSource.scan`. The host forces `source`
to the plugin id and, until the library stores works, expressions and
editions, maps the record onto one flat library row.

**Fields**

- `account_id`: Account that owns this library entry.
- `product_id`: Storefront product / SKU id.
- `title`: Primary title string.
- `marketplace`: Marketplace / region when known.
  `None` when absent (wire zero value).
- `publisher`: Publisher name when known.
  `None` when absent (wire zero value).
- `length_minutes`: Runtime in whole minutes when known.
  `None` when absent (wire zero value).
- `subtitle`: Subtitle when distinct from `title`.
  `None` when absent (wire zero value).
- `work_kind`: Kind of work the title is.
- `expression_kind`: How the work is realised in this edition.
- `media_kind`: Carrier of this edition.
- `identifiers`: External identifiers (ISBN, ASIN, OpenLibrary ids).
- `credits`: Credited contributors in billing order.
- `series_entries`: Series the work belongs to.
- `parent_product_id`: Product id of the parent podcast show, for a podcast episode.
  `None` when absent (wire zero value).
- `language`: Content language code when known.
  `None` when absent (wire zero value).
- `abridgement`: Whether the edition is abridged when the storefront says so.
- `published_at`: Publication date as RFC 3339 when known.
  `None` when absent (wire zero value).
- `purchased_at`: When the account bought or added the title, as RFC 3339, when known.
  `None` when absent (wire zero value).
- `categories`: Category / genre labels as a single string when known.
  `None` when absent (wire zero value).
- `explicitness`: Whether the storefront marks the edition explicit (adult) content.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ScanPage

```rust
pub struct ScanPage {
    pub books: Vec<ScanBook>,
    pub removed: Vec<String>,
    pub next_cursor: String,
    pub sync_token: String,
    pub complete: bool,
    pub store_pages: u32,
    pub refreshed_credentials: Option<Vec<u8>>,
}
```

One page of an account's scan pass, the result of `ContentSource.scan`.
A pass is the run of pages from an empty `cursor` to the page whose
`nextCursor` is empty; `syncToken` and `complete` are read from that last
page only. A guest that does not page answers the first call with every
title, an empty `nextCursor` and `complete = true`.

**Fields**

- `books`: Titles for the host to upsert; at most `maxScanPageBooks`.
- `removed`: Product ids the storefront reports as gone from the account since the
  pass's `syncToken` (delta passes only); at most `maxScanPageBooks`.
- `next_cursor`: Opaque position of the next page; empty ends the pass. At most
  `maxScanCursorBytes`; never equal to the cursor this page answered.
- `sync_token`: On the last page: the opaque token the host sends as
  `ScanParams.syncToken` on the account's next pass; empty means the next
  pass is a full one. At most `maxScanCursorBytes`.
- `complete`: On the last page: true when the pass listed every title the account
  holds (a full pass), false for a delta pass that listed only changes.
- `store_pages`: Storefront requests this page took.
- `refreshed_credentials`: The account's credentials as the guest holds them after refreshing them
  while producing this page (for example a renewed access token): the
  same encoding login returned, a JSON object of at most
  `maxRefreshedCredentialBytes`. The host stores them in place of the
  credentials it sent, unless the account was signed in again meanwhile,
  and sends them on the pass's later pages. Empty when nothing changed.
  `None` when absent (wire zero value).

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

### FetchedFile

```rust
pub struct FetchedFile {
    pub path: String,
    pub role: FetchedFileRole,
    pub content_type: Option<String>,
    pub title: Option<String>,
    pub duration_ms: Option<u64>,
}
```

One file `fetchTitle` wrote under the cache directory.

**Fields**

- `path`: Absolute path to the file under `cacheDir`.
- `role`: What the file is to the edition.
- `content_type`: MIME type when the guest knows it (`audio/mp4`, `application/epub+zip`,
  `image/jpeg`); the host sniffs the file either way.
  `None` when absent (wire zero value).
- `title`: Part title (disc or chapter label), for a `part`.
  `None` when absent (wire zero value).
- `duration_ms`: Playing time of the file in milliseconds, for audio, when known.
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
    pub media_kind: MediaKind,
    pub files: Vec<FetchedFile>,
    pub chapters: Vec<ChapterMarker>,
    pub pdf_url: Option<String>,
    pub refreshed_credentials: Option<Vec<u8>>,
    pub chapter_kind: ChapterKind,
}
```

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

**Fields**

- `media_kind`: Carrier of what was fetched. Must be one of the storefront's declared
  `MediaSupport.fetches`; `unspecified` is refused.
- `files`: The files the guest wrote under the cache directory: the edition's
  ordered `part`s or one `whole` file, plus an optional `cover`.
- `chapters`: Chapter markers; empty when unknown.
- `pdf_url`: Companion PDF download URL when the store exposes one; the host
  downloads it itself.
  `None` when absent (wire zero value).
- `refreshed_credentials`: The account's credentials as the guest holds them after refreshing them
  during the fetch, as in `ScanPage.refreshedCredentials`. Empty when
  nothing changed.
  `None` when absent (wire zero value).
- `chapter_kind`: What `chapters` marks. The host ranks literary chapters above track
  boundaries when it decides an edition's chapters.

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
- `limit`: Maximum hits to return; 0 means the storefront default (20 hits).
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
    pub lookup: PriceLookup,
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
- `lookup`: How the host asks the store to find the title. `unspecified` (an older
  host) leaves the guest free to use any field, as before.

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

### RevokeAccountParams

```rust
pub struct RevokeAccountParams {
    pub account_id: String,
    pub credentials: Option<Vec<u8>>,
}
```

Params of `ContentSource.revokeAccount`. The host sends the credentials it
is about to delete so the guest can sign the device out at the store.

**Fields**

- `account_id`: Account being revoked.
- `credentials`: Decrypted credential blob for this account, exactly as login returned
  it; never log it. Empty when none is stored.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CatalogHit

```rust
pub struct CatalogHit {
    pub product_id: String,
    pub title: String,
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
    pub work_kind: WorkKind,
    pub expression_kind: ExpressionKind,
    pub media_kind: MediaKind,
    pub identifiers: Vec<Identifier>,
    pub credits: Vec<Credit>,
    pub series_entries: Vec<SeriesEntry>,
    pub parent_product_id: Option<String>,
    pub rank_hint: Option<u32>,
    pub explicitness: Explicitness,
    pub sample_url: Option<String>,
}
```

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

**Fields**

- `product_id`: Storefront product / SKU id.
- `title`: Primary title.
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
- `work_kind`: Kind of work the hit is.
- `expression_kind`: How the work is realised in this edition.
- `media_kind`: Carrier of this edition.
- `identifiers`: External identifiers (ISBN, ASIN, OpenLibrary ids).
- `credits`: Credited contributors in billing order.
- `series_entries`: Series the work belongs to.
- `parent_product_id`: Product id of the parent podcast show, for a podcast episode.
  `None` when absent (wire zero value).
- `rank_hint`: The storefront's own rank for this hit in `searchCatalog` results:
  1-based position in the store's ordering for the requested sort across
  all pages (page 2 of 20-hit pages starts at 21), or a store-wide
  popularity rank when the store has one. 0 means unranked. Catalog search
  merges stores by it for relevance and popularity sorts and puts unranked
  hits after ranked ones. Other catalog methods leave it 0.
  `None` when absent (wire zero value).
- `explicitness`: Whether the storefront marks the edition explicit (adult) content.
- `sample_url`: Absolute `https://` URL of a short audio sample of this edition that
  the storefront offers publicly (a few minutes, in any format the media
  worker decodes). The host never hands it to a client: a book's details
  play it through a same-origin path, fetched by a `covers` plugin with
  `media = audio` and re-encoded in the media worker. Empty when the store
  has none.
  `None` when absent (wire zero value).

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
    pub records: Vec<PricedRecord>,
}
```

Success payload of `ContentSource.purchaseHint`.

**Fields**

- `found`: False when the guest could not resolve the title (`hint` is empty).
- `hint`: Purchase hint when `found`.
- `records`: Every priced listing a title search matched (`lookup = title`), each
  with its full catalog record, so the host can tell recordings apart by
  identifiers, credits and kinds rather than by title. Empty for other
  lookups and for guests that do not return records; `hint` then stays
  the answer.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### PricedRecord

```rust
pub struct PricedRecord {
    pub record: CatalogHit,
    pub hint: PurchaseHint,
}
```

One priced listing returned by a title search.

**Fields**

- `record`: The listing's catalog record: identifiers, credits, kinds, language.
- `hint`: Its purchase link and prices, as `PurchaseHint` carries them.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### PurchaseHintsParams

```rust
pub struct PurchaseHintsParams {
    pub queries: Vec<PurchaseHintParams>,
}
```

Params of `ContentSource.purchaseHints`: the titles to price together.

**Fields**

- `queries`: One query per title, each as for `purchaseHint`; at most `maxListPage`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PurchaseHintsResult

```rust
pub struct PurchaseHintsResult {
    pub results: Vec<PurchaseHintResult>,
}
```

Success payload of `ContentSource.purchaseHints`.

**Fields**

- `results`: One answer per query, in the order the queries were sent.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### QualityOffer

```rust
pub struct QualityOffer {
    pub codec: OfferedCodec,
    pub aac_profile: OfferedAacProfile,
    pub bitrate_kbps: Option<u32>,
    pub sample_rate_hz: Option<u32>,
    pub channels: Option<u32>,
}
```

The stream one account would download for a title today.

**Fields**

- `codec`: Codec of the offered stream.
- `aac_profile`: AAC profile when `codec` is `aac`; `unknown` otherwise.
- `bitrate_kbps`: Average bitrate in kbit/s; zero when unknown.
  `None` when absent (wire zero value).
- `sample_rate_hz`: Sample rate in Hz; zero when unknown.
  `None` when absent (wire zero value).
- `channels`: Channel count; zero when unknown.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QualityOffersParams

```rust
pub struct QualityOffersParams {
    pub account_id: String,
    pub credentials: Option<Vec<u8>>,
    pub source_config: ExtensibleConfig,
    pub title_ids: Vec<String>,
    pub fetch: FetchOptions,
}
```

Params of `ContentSource.qualityOffers`: the titles one account holds.

**Fields**

- `account_id`: Account whose download would be made.
- `credentials`: Decrypted credential blob for this account; never log it. Empty when
  none is stored.
  `None` when absent (wire zero value).
- `source_config`: The instance's settings as `application/json` (bitrate tier and the
  like, which change what a download gets).
- `title_ids`: Store product ids, as the account's scan listed them; at most
  `maxListPage`.
- `fetch`: The fetch options a download would carry (DRM path, xHE-AAC), so the
  answer names what that download would get.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TitleQualityOffer

```rust
pub struct TitleQualityOffer {
    pub found: bool,
    pub offer: QualityOffer,
}
```

One title's answer in `QualityOffersResult`.

**Fields**

- `found`: False when the store could not say for this title (`offer` is empty).
- `offer`: What a download would get, when `found`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### QualityOffersResult

```rust
pub struct QualityOffersResult {
    pub offers: Vec<TitleQualityOffer>,
    pub refreshed_credentials: Option<Vec<u8>>,
}
```

Success payload of `ContentSource.qualityOffers`.

**Fields**

- `offers`: One answer per title id, in the order they were sent. A short list
  leaves the rest unanswered.
- `refreshed_credentials`: The account's credentials as the guest holds them after refreshing them
  during the call, as in `ScanPage.refreshedCredentials`. Empty when
  nothing changed.
  `None` when absent (wire zero value).

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
    pub purpose: InvocationPurpose,
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
- `purpose`: What the host opens the entrypoints for. `provision` opens an instance
  that may not be configured yet: `CONFIG` and `SECRETS` hold what is
  stored so far, required settings may be missing, and the host calls
  only `Entrypoints.provisioning`.

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

### ScheduledController

```rust
pub struct ScheduledController {
    pub cron: String,
    pub scheduled_time: u64,
    pub fire_id: String,
    pub attempt: u32,
    pub deadline_unix_ms: u64,
}
```

One fire of a `\[triggers\] crons` schedule (`scheduled(controller)`).

**Fields**

- `cron`: The schedule that fired, normalized to single spaces (`0 3 * * MON`).
- `scheduled_time`: Fire time, UTC unix milliseconds (always a whole minute). Retries of
  one fire keep it.
- `fire_id`: Durable id of this fire, stable across retries; dedupe on it.
- `attempt`: Failure attempt, starting at 1.
- `deadline_unix_ms`: UTC unix milliseconds. Host fence/lease is authoritative; this hint must
  not outlive the fence.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataLookupParams

```rust
pub struct MetadataLookupParams {
    pub identifiers: Vec<Identifier>,
    pub marketplace: Option<String>,
    pub limit: Option<u32>,
}
```

Params of `MetadataProvider.lookup`.

**Fields**

- `identifiers`: Identifiers naming the wanted edition or work; never empty. A provider
  ignores schemes it does not serve.
- `marketplace`: Marketplace to answer for (`us`, `uk`, ...) when the provider is
  regional.
  `None` when absent (wire zero value).
- `limit`: Most records to return; the provider picks a small default when absent.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataSearchParams

```rust
pub struct MetadataSearchParams {
    pub title: String,
    pub author: Option<String>,
    pub language: Option<String>,
    pub marketplace: Option<String>,
    pub limit: Option<u32>,
}
```

Params of `MetadataProvider.search`.

**Fields**

- `title`: Title to match; never empty.
- `author`: Author display name to narrow the match when known.
  `None` when absent (wire zero value).
- `language`: Preferred content language code when known.
  `None` when absent (wire zero value).
- `marketplace`: Marketplace to answer for when the provider is regional.
  `None` when absent (wire zero value).
- `limit`: Most records to return; the provider picks a small default when absent.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataChaptersParams

```rust
pub struct MetadataChaptersParams {
    pub identifiers: Vec<Identifier>,
    pub marketplace: Option<String>,
}
```

Params of `MetadataProvider.chapters`.

**Fields**

- `identifiers`: Identifiers of one edition (an ASIN for Audnexus); never empty.
- `marketplace`: Marketplace to answer for when the provider is regional.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataRecord

```rust
pub struct MetadataRecord {
    pub title: String,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub cover_url: Option<String>,
    pub url: Option<String>,
    pub publisher: Option<String>,
    pub length_minutes: Option<u32>,
    pub published_at: Option<String>,
    pub categories: Option<String>,
    pub language: Option<String>,
    pub abridgement: Abridgement,
    pub work_kind: WorkKind,
    pub expression_kind: ExpressionKind,
    pub media_kind: MediaKind,
    pub identifiers: Vec<Identifier>,
    pub credits: Vec<Credit>,
    pub series_entries: Vec<SeriesEntry>,
    pub rating_overall: Option<f64>,
    pub rating_count: Option<u32>,
}
```

One edition as a metadata provider describes it: the same structured
fields as `CatalogHit`, without store pricing.

**Fields**

- `title`: Primary title.
- `subtitle`: Subtitle when distinct from `title`.
  `None` when absent (wire zero value).
- `description`: Long description / blurb when known.
  `None` when absent (wire zero value).
- `cover_url`: Cover image URL.
  `None` when absent (wire zero value).
- `url`: The provider's page for this record.
  `None` when absent (wire zero value).
- `publisher`: Publisher name when known.
  `None` when absent (wire zero value).
- `length_minutes`: Runtime in whole minutes when known.
  `None` when absent (wire zero value).
- `published_at`: Publication date string as the provider prints it.
  `None` when absent (wire zero value).
- `categories`: Category / genre labels as a single string when known.
  `None` when absent (wire zero value).
- `language`: Content language code when known.
  `None` when absent (wire zero value).
- `abridgement`: Whether the edition is abridged when the provider says so.
- `work_kind`: Kind of work the record describes.
- `expression_kind`: How the work is realised in this edition.
- `media_kind`: Carrier of this edition.
- `identifiers`: External identifiers (ISBN, ASIN, OpenLibrary ids).
- `credits`: Credited contributors in billing order.
- `series_entries`: Series the work belongs to.
- `rating_overall`: Aggregate rating when the provider publishes one.
  `None` when absent (wire zero value).
- `rating_count`: Number of ratings behind `ratingOverall` when known.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### MetadataRecords

```rust
pub struct MetadataRecords {
    pub records: Vec<MetadataRecord>,
}
```

Success payload of `MetadataProvider.lookup` / `search`.

**Fields**

- `records`: Records in the provider's relevance order; empty when nothing matched.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `PartialEq`, `Serialize`

### MetadataChapter

```rust
pub struct MetadataChapter {
    pub title: String,
    pub start_ms: u64,
    pub length_ms: Option<u64>,
    pub depth: u32,
}
```

One chapter in a provider's chapter layout. Chapters are listed in
pre-order: a chapter with `depth` n + 1 is nested under the nearest
earlier chapter with `depth` n.

**Fields**

- `title`: Chapter title.
- `start_ms`: Start offset in milliseconds from the beginning of the title.
- `length_ms`: Length in milliseconds when known.
  `None` when absent (wire zero value).
- `depth`: Nesting depth; 0 for a top-level chapter.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataChapters

```rust
pub struct MetadataChapters {
    pub found: bool,
    pub chapters: Vec<MetadataChapter>,
    pub brand_intro_ms: Option<u64>,
    pub brand_outro_ms: Option<u64>,
    pub runtime_ms: Option<u64>,
    pub accurate: bool,
}
```

Success payload of `MetadataProvider.chapters`.

**Fields**

- `found`: False when the provider has no chapter layout for the edition.
- `chapters`: Chapters in pre-order; empty when not `found`.
- `brand_intro_ms`: Length of the store's spoken brand intro in milliseconds when known.
  `None` when absent (wire zero value).
- `brand_outro_ms`: Length of the store's spoken brand outro in milliseconds when known.
  `None` when absent (wire zero value).
- `runtime_ms`: Total runtime the layout was measured against, in milliseconds.
  `None` when absent (wire zero value).
- `accurate`: Whether the provider marks the offsets as exact rather than estimated.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CoverFetchParams

```rust
pub struct CoverFetchParams {
    pub urls: Vec<String>,
    pub max_bytes: u32,
    pub conditions: Vec<CoverCondition>,
    pub media: RemoteMedia,
}
```

Params of `CoverFetcher.fetchCovers`.

**Fields**

- `urls`: Absolute `https://` URLs to fetch, at most `maxCoverUrls`; never empty.
  The host has already checked each one against its URL policy.
- `max_bytes`: Largest body, in bytes, the host accepts for one image; at most
  `maxCoverBytes`. The sink refuses a larger body, which fails that URL
  only.
- `conditions`: Conditional request headers per URL, in the same order as `urls`, from
  the host's cached copy. Empty, or shorter than `urls`, means the
  remaining URLs are fetched unconditionally.
- `media`: What the URLs name: cover art (the default) or a storefront's audio
  sample (`CatalogHit.sampleUrl`). A fetcher sends a matching `Accept`
  header and refuses a body whose `Content-Type` names something else;
  the host checks and re-encodes either kind regardless.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CoverCondition

```rust
pub struct CoverCondition {
    pub if_none_match: Option<String>,
    pub if_modified_since: Option<String>,
}
```

Validators the host already holds for one URL, sent as conditional request
headers so an unchanged image costs a `304` instead of its body.

**Fields**

- `if_none_match`: `If-None-Match` value: the `ETag` the host stored, verbatim; empty for
  none.
  `None` when absent (wire zero value).
- `if_modified_since`: `If-Modified-Since` value: the `Last-Modified` the host stored,
  verbatim; empty for none.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CoverImage

```rust
pub struct CoverImage {
    pub url: String,
    pub stored: bool,
    pub content_type: Option<String>,
    pub error: Option<String>,
    pub not_modified: bool,
    pub etag: Option<String>,
    pub last_modified: Option<String>,
    pub cache_control: Option<String>,
    pub expires: Option<String>,
    pub date: Option<String>,
    pub age: Option<String>,
}
```

What happened to one requested URL.

**Fields**

- `url`: The requested URL, verbatim.
- `stored`: Whether the body was put into the sink under this URL's index.
- `content_type`: `Content-Type` the server sent. Advisory: the host sniffs the bytes.
  `None` when absent (wire zero value).
- `error`: Why nothing was stored (HTTP status, size cap, timeout, refused host).
  `None` when absent (wire zero value).
- `not_modified`: The server answered `304 Not Modified` to the URL's conditions: nothing
  was put into the sink and the host's copy is still current.
- `etag`: `ETag` the server sent, verbatim. Advisory: the host parses and caps it.
  `None` when absent (wire zero value).
- `last_modified`: `Last-Modified` the server sent, verbatim.
  `None` when absent (wire zero value).
- `cache_control`: `Cache-Control` the server sent, verbatim (several lines joined with
  `, `).
  `None` when absent (wire zero value).
- `expires`: `Expires` the server sent, verbatim.
  `None` when absent (wire zero value).
- `date`: `Date` the server sent, verbatim.
  `None` when absent (wire zero value).
- `age`: `Age` the server (or a cache in front of it) sent, verbatim.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### CoverImages

```rust
pub struct CoverImages {
    pub images: Vec<CoverImage>,
}
```

Success payload of `CoverFetcher.fetchCovers`.

**Fields**

- `images`: One entry per requested URL, in request order.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataFileRecord

```rust
pub struct MetadataFileRecord {
    pub present: Vec<MetadataField>,
    pub title: Option<String>,
    pub subtitle: Option<String>,
    pub description: Option<String>,
    pub publisher: Option<String>,
    pub published_at: Option<String>,
    pub language: Option<String>,
    pub genres: Vec<String>,
    pub tags: Vec<String>,
    pub abridgement: Abridgement,
    pub expression_kind: ExpressionKind,
    pub identifiers: Vec<Identifier>,
    pub credits: Vec<Credit>,
    pub credit_roles: Vec<CreditRole>,
    pub series_entries: Vec<SeriesEntry>,
    pub chapters: Vec<MetadataChapter>,
    pub duration_ms: Option<u64>,
    pub explicitness: Explicitness,
}
```

What one metadata file says about the edition it sits beside. A field the
file does not state is left out of `present` and means "unknown", never
"cleared"; a field in `present` with an empty value means the file states
that it is empty. The host checks every value against the `maxMetadataFile*`
caps and refuses a record that states a field its dialect does not carry.

**Fields**

- `present`: Fields the file states, each at most once; at most the fields the
  dialect declares.
- `title`: Primary title.
  `None` when absent (wire zero value).
- `subtitle`: Subtitle when distinct from `title`.
  `None` when absent (wire zero value).
- `description`: Long description / blurb; at most `maxMetadataFileDescriptionBytes`.
  `None` when absent (wire zero value).
- `publisher`: Publisher name.
  `None` when absent (wire zero value).
- `published_at`: Publication date as `YYYY`, `YYYY-MM` or `YYYY-MM-DD`.
  `None` when absent (wire zero value).
- `language`: Content language as the file prints it (a code or a name).
  `None` when absent (wire zero value).
- `genres`: Genre labels in file order.
- `tags`: Free tags in file order.
- `abridgement`: Whether the edition is abridged; `unknown` when the file does not say.
- `expression_kind`: How the work is realised; `unspecified` when the file does not say.
- `identifiers`: External identifiers the file carries.
- `credits`: Credited contributors in billing order, only in the roles `creditRoles`
  names.
- `credit_roles`: Credit roles the file states, each at most once. A role listed here
  with no credit means the file states nobody in that role; a role left
  out is unknown. Empty unless `present` holds `credits`.
- `series_entries`: Series the work belongs to.
- `chapters`: Chapter layout in pre-order (see `MetadataChapter`); at most
  `maxMetadataFileChapters`, starts never decreasing.
- `duration_ms`: Running time in milliseconds.
  `None` when absent (wire zero value).
- `explicitness`: Whether the edition is explicit (adult) content; `unknown` when the
  file does not say.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataFileParseParams

```rust
pub struct MetadataFileParseParams {
    pub dialect: String,
    pub file_name: String,
    pub bytes: Vec<u8>,
}
```

Params of `MetadataFiles.parse`.

**Fields**

- `dialect`: Dialect id from the manifest's `\[\[metadata_files.dialects\]\]` that the
  host routed the file to.
- `file_name`: The file's name without any folder (`metadata.json`, `Book.opf`).
- `bytes`: The whole file; at most `maxMetadataFileBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataFileParsed

```rust
pub struct MetadataFileParsed {
    pub record: MetadataFileRecord,
}
```

Success payload of `MetadataFiles.parse`.

**Fields**

- `record`: What the file says.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataAudioFile

```rust
pub struct MetadataAudioFile {
    pub name: String,
    pub duration_ms: u64,
}
```

One audio file of the edition a sidecar describes, for formats that name
their audio (a CUE sheet's `FILE` lines).

**Fields**

- `name`: The file's name without any folder, as it sits next to the sidecar.
- `duration_ms`: Running time in milliseconds; 0 when the host does not know it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataFileRenderParams

```rust
pub struct MetadataFileRenderParams {
    pub dialect: String,
    pub record: MetadataFileRecord,
    pub fields: Vec<MetadataField>,
    pub existing: Vec<u8>,
    pub audio_files: Vec<MetadataAudioFile>,
}
```

Params of `MetadataFiles.render`.

**Fields**

- `dialect`: Dialect id from the manifest; one that declares `writes = true`.
- `record`: Values to write. Only the fields in `fields` are read from it.
- `fields`: Fields to set, each at most once and all declared by the dialect. A
  listed field with an empty value removes it from the file.
- `existing`: The file as it is now; empty when the host is creating it. Everything
  the dialect does not change (unknown keys, comments, other fields) is
  kept. At most `maxMetadataFileBytes`.
- `audio_files`: The edition's audio files in playback order, at most
  `maxMetadataAudioFiles`; empty when the host has none or does not know
  them. Chapter starts in `record` count from the start of the first file.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### MetadataFileRendered

```rust
pub struct MetadataFileRendered {
    pub bytes: Vec<u8>,
}
```

Success payload of `MetadataFiles.render`.

**Fields**

- `bytes`: The new file; at most `maxMetadataFileBytes`. Empty when the dialect has
  nothing left to write; the host decides whether to remove the file.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotificationField

```rust
pub struct NotificationField {
    pub name: String,
    pub value: String,
}
```

One labelled value shown with a notification.

**Fields**

- `name`: Label; at most `maxNotificationTitleBytes`.
- `value`: Value; at most `maxNotificationFieldBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotificationMessage

```rust
pub struct NotificationMessage {
    pub notification_id: String,
    pub notification_type: String,
    pub severity: NotificationSeverity,
    pub title: String,
    pub text: String,
    pub markdown: Option<String>,
    pub html: Option<String>,
    pub link: Option<String>,
    pub fields: Vec<NotificationField>,
    pub group_key: Option<String>,
    pub alert_action: NotificationAlertAction,
    pub alert_key: Option<String>,
    pub sensitive: bool,
}
```

A message the host rendered for one notification. Every link is built
from the library's public origin, never from a request.

**Fields**

- `notification_id`: Host-assigned id, the same on every destination's copy; plugins use it
  as an idempotency key where the service takes one.
- `notification_type`: Notification type id (`job_failed`, `<plugin>:sync_failed`).
- `severity`: How urgent it is.
- `title`: Short title; at most `maxNotificationTitleBytes`.
- `text`: Plain-text body; at most `maxNotificationTextBytes`.
- `markdown`: Markdown body for chat services; at most `maxNotificationTextBytes`.
  `None` when absent (wire zero value).
- `html`: Minimal HTML body for email; at most `maxNotificationHtmlBytes`.
  `None` when absent (wire zero value).
- `link`: Absolute `https` (or loopback `http`) link into the library.
  `None` when absent (wire zero value).
- `fields`: Labelled values, at most `maxNotificationFields`.
- `group_key`: Messages with one group key may be threaded or collapsed together.
  `None` when absent (wire zero value).
- `alert_action`: Alert lifecycle step; `none` for a one-off message.
- `alert_key`: Alert this message opens or closes; required unless `alertAction` is
  `none`.
  `None` when absent (wire zero value).
- `sensitive`: True when the body carries a secret (an account-mail link): the plugin
  and SDKs must never log it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyDestination

```rust
pub struct NotifyDestination {
    pub kind: NotifyDestinationKind,
    pub destination_id: String,
    pub address_json: String,
}
```

Where a `Notifier.send` goes.

**Fields**

- `kind`: Channel or personal destination.
- `destination_id`: Host id of the destination, stable across sends.
- `address_json`: Personal address as a JSON object of the fields the manifest's
  `\[\[notifier.address_fields\]\]` declare; empty for a channel. At most
  `maxNotifierAddressBytes`. May hold secrets; never logged.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifySendParams

```rust
pub struct NotifySendParams {
    pub message: NotificationMessage,
    pub destination: NotifyDestination,
}
```

Params of `Notifier.send`.

**Fields**

- `message`: What to send.
- `destination`: Where to send it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifySent

```rust
pub struct NotifySent {
    pub provider_id: Option<String>,
}
```

Success payload of `Notifier.send`.

**Fields**

- `provider_id`: The service's id for the delivered message, when it gives one.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyValidateParams

```rust
pub struct NotifyValidateParams {
    pub address_json: String,
}
```

Params of `Notifier.validateDestination`.

**Fields**

- `address_json`: Address document as the patron entered it; at most
  `maxNotifierAddressBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyValidated

```rust
pub struct NotifyValidated {
    pub address_json: String,
    pub label: Option<String>,
}
```

Success payload of `Notifier.validateDestination`.

**Fields**

- `address_json`: Normalised address document the host saves; at most
  `maxNotifierAddressBytes`.
- `label`: Short label shown to the patron (`a***@example.com`); never a secret.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyRaiseParams

```rust
pub struct NotifyRaiseParams {
    pub notification_type: String,
    pub fields: Vec<NotificationField>,
    pub patron_id: Option<String>,
    pub link_path: Option<String>,
    pub group_key: Option<String>,
    pub alert_action: NotificationAlertAction,
    pub alert_key: Option<String>,
}
```

Params of `NotificationRaiser.raise`.

**Fields**

- `notification_type`: Type id from the plugin's `\[\[notifications.types\]\]` (without the plugin
  prefix the host adds).
- `fields`: Labelled values, at most `maxNotificationFields`.
- `patron_id`: Patron the notification is for; required for a patron-audience type
  and must be inside the invocation's account scope; empty otherwise.
  `None` when absent (wire zero value).
- `link_path`: Path inside the library the notification links to (starts with `/`);
  the host prefixes the public origin.
  `None` when absent (wire zero value).
- `group_key`: Messages with one group key may be threaded or collapsed together.
  `None` when absent (wire zero value).
- `alert_action`: Alert lifecycle step; `none` for a one-off notification.
- `alert_key`: Alert this raise opens or closes; required unless `alertAction` is
  `none`.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### NotifyRaised

```rust
pub struct NotifyRaised {
    pub notification_id: String,
}
```

Success payload of `NotificationRaiser.raise`.

**Fields**

- `notification_id`: Host-assigned notification id.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiModel

```rust
pub struct AiModel {
    pub model_id: String,
    pub task: AiTask,
    pub revision: String,
    pub dimensions: Option<u32>,
    pub max_tokens: Option<u32>,
    pub languages: Vec<String>,
}
```

One model an `ai` plugin serves.

**Fields**

- `model_id`: Model id within the plugin (`bekko-a8m`): lowercase letters, digits,
  `.`, `_` and `-`. Callers name it `@<plugin id>/<model id>`; the whole
  name is at most `maxAiModelNameBytes`.
- `task`: The model's one task.
- `revision`: Digest of everything that decides the output: weights, tokenizer and
  the language packs loaded. Outputs are comparable only when it matches.
- `dimensions`: Text embeddings: numbers per vector, at most `maxAiEmbedDimensions`.
  `None` when absent (wire zero value).
- `max_tokens`: Text embeddings: tokens read from one text; the rest is cut off.
  `None` when absent (wire zero value).
- `languages`: BCP 47 tags of the languages the model is loaded for, at most
  `maxAiLanguages`; empty when it has no language packs.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiModelList

```rust
pub struct AiModelList {
    pub models: Vec<AiModel>,
}
```

Success payload of `AiModels.models`.

**Fields**

- `models`: Served models, at most `maxAiModels`, each `modelId` once.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiEmbedText

```rust
pub struct AiEmbedText {
    pub text: String,
    pub role: AiTextRole,
}
```

One text to embed.

**Fields**

- `text`: The text, at most `maxAiEmbedTextBytes`; never empty.
- `role`: Document or query.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiTextEmbeddingsInput

```rust
pub struct AiTextEmbeddingsInput {
    pub texts: Vec<AiEmbedText>,
}
```

Input of the `textEmbeddings` task.

**Fields**

- `texts`: Texts to embed, 1 to `maxAiEmbedTexts`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiTextEmbeddingsOutput

```rust
pub struct AiTextEmbeddingsOutput {
    pub dimensions: u32,
    pub vectors: Vec<u8>,
    pub revision: String,
    pub fallbacks: Vec<u32>,
}
```

Output of the `textEmbeddings` task.

**Fields**

- `dimensions`: Numbers per vector; equals the model's `AiModel.dimensions`.
- `vectors`: One vector per input text, in input order: `dimensions` little-endian
  IEEE 754 32-bit floats each, concatenated. Every number is finite.
- `revision`: `AiModel.revision` of the model that made the vectors.
- `fallbacks`: Indexes (into the input texts, ascending, each once) of the texts
  where the tokenizer met text outside the loaded language packs and
  fell back to smaller pieces; such a vector should be recomputed after
  the missing pack is added.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiInput

```rust
pub struct AiInput {
    pub text_embeddings: AiTextEmbeddingsInput,
}
```

Input to one model run: one field per `AiTask`, and exactly one is set
(the others are null). A new task appends a field.

**Fields**

- `text_embeddings`: `textEmbeddings` task.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiOutput

```rust
pub struct AiOutput {
    pub text_embeddings: AiTextEmbeddingsOutput,
}
```

Output of one model run: one field per `AiTask`, and exactly one is set
(the others are null). A new task appends a field.

**Fields**

- `text_embeddings`: `textEmbeddings` task.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiRunParams

```rust
pub struct AiRunParams {
    pub model: String,
    pub input: AiInput,
}
```

Params of `AiModels.run` and `AiRunner.run`.

**Fields**

- `model`: On `AiModels.run`, the plugin's own `AiModel.modelId`. On
  `AiRunner.run`, `@<plugin id>/<model id>` or a task alias
  (`@library/text-embeddings`: the model the library chose for the task).
- `input`: The input; the field set must match the model's task.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### AiRunOk

```rust
pub struct AiRunOk {
    pub output: AiOutput,
    pub model: Option<String>,
}
```

Success payload of a model run.

**Fields**

- `output`: The output; the field set matches the model's task.
- `model`: Full name of the model that ran (`@<plugin id>/<model id>`), so a
  caller that used a task alias knows what answered. Set by the host on
  `AiRunner.run`; empty on `AiModels.run`.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketHeader

```rust
pub struct SocketHeader {
    pub name: String,
    pub value: String,
}
```

One request header on `Sockets.connect`.

**Fields**

- `name`: Header name; the broker refuses hop-by-hop and WebSocket handshake names.
- `value`: Header value. May carry a credential: the host never logs it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketAutoResponse

```rust
pub struct SocketAutoResponse {
    pub request: SocketFrame,
    pub response: SocketFrame,
}
```

A heartbeat the broker answers itself: when `request` arrives byte for byte,
it sends `response` and does not wake the guest.

**Fields**

- `request`: Frame to match, at most `maxSocketAutoResponseBytes`.
- `response`: Frame to send back, at most `maxSocketAutoResponseBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketConnectRequest

```rust
pub struct SocketConnectRequest {
    pub url: String,
    pub protocols: Vec<String>,
    pub headers: Vec<SocketHeader>,
    pub auto_responses: Vec<SocketAutoResponse>,
    pub attachment: Vec<u8>,
    pub trust_anchors: Vec<Vec<u8>>,
    pub server_name: String,
}
```

Arguments of `Sockets.connect`.

**Fields**

- `url`: `wss://` or `ws://` URL, at most `maxSocketUrlBytes`, or
  `tls://host:port` for a raw TLS byte stream (port required, nothing
  after it). The host must be on the plugin's network grant; a `tls://`
  stream needs a `tcp` grant for that host and port.
- `protocols`: Requested subprotocols, at most `maxSocketHeaders`; on a `tls://`
  stream, the ALPN protocol ids to offer (visible ASCII, 1 to 255 bytes).
- `headers`: Extra request headers, at most `maxSocketHeaders`; none on `tls://`.
- `auto_responses`: Heartbeats the broker answers, at most `maxSocketAutoResponses`; none on
  `tls://`.
- `attachment`: Initial attachment, at most `maxSocketAttachmentBytes`.
- `trust_anchors`: `tls://` only: DER root certificates, at most `maxSocketTrustAnchors` of
  at most `maxSocketTrustAnchorBytes` each, that the server's chain must
  end in instead of Mozilla's roots, for a service that signs its own
  (an ngrok or Cloudflare Tunnel agent endpoint). Empty uses Mozilla's.
- `server_name`: `tls://` only: the DNS name to send as SNI and to check the server's
  certificate against, for an endpoint reached at one name that answers
  for another (a Cloudflare Tunnel edge, dialed at `region1.v2.argotunnel.com`
  and serving `h2.cftunnel.com`). It must be on a `tcp` grant for the
  URL's port too, so it can't route past the grant by name. At most
  `maxSocketServerNameBytes`; empty uses the URL's host.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketInfo

```rust
pub struct SocketInfo {
    pub socket: String,
    pub url: String,
    pub protocol: Option<String>,
    pub attachment: Vec<u8>,
}
```

One open host-brokered socket.

**Fields**

- `socket`: Host-minted socket id, stable for the socket's life.
- `url`: URL the socket was opened to, without credentials.
- `protocol`: Subprotocol the server chose (on `tls://`, the negotiated ALPN id);
  empty when none.
  `None` when absent (wire zero value).
- `attachment`: Current attachment.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketList

```rust
pub struct SocketList {
    pub sockets: Vec<SocketInfo>,
}
```

Success payload of `Sockets.list`.

**Fields**

- `sockets`: Open sockets, oldest first.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketMessageOk

```rust
pub struct SocketMessageOk {
    pub poll_events: bool,
}
```

What a guest asks of the host after handling one pushed message.

**Fields**

- `poll_events`: True asks the host to call `Integration.pollEvents` now, while the
  guest is awake, instead of at the next scheduled poll: the message
  announced something the poll hands over (a new user). The host ignores
  it for a plugin whose host schedules no poll.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketMessage

```rust
pub struct SocketMessage {
    pub socket: String,
    pub frame: SocketFrame,
    pub attachment: Vec<u8>,
}
```

Argument of `SocketHandler.message`.

**Fields**

- `socket`: Socket the message arrived on.
- `frame`: The message.
- `attachment`: The socket's attachment when the message arrived.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketClosed

```rust
pub struct SocketClosed {
    pub socket: String,
    pub code: u16,
    pub reason: Option<String>,
    pub clean: bool,
    pub attachment: Vec<u8>,
}
```

Argument of `SocketHandler.closed`.

**Fields**

- `socket`: Socket that closed; its id is not reused.
- `code`: WebSocket close code; 1006 when the connection dropped without one.
- `reason`: Close reason the peer sent; empty when none.
  `None` when absent (wire zero value).
- `clean`: True when the close handshake completed.
- `attachment`: The socket's last attachment.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OriginHeader

```rust
pub struct OriginHeader {
    pub name: String,
    pub value: String,
}
```

One header field on an `Origin.fetch` request or response.

**Fields**

- `name`: Field name, an HTTP token; the host refuses hop-by-hop names.
- `value`: Field value, visible ASCII, spaces and tabs. May carry a credential (a
  cookie, an authorization): the host never logs it.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OriginRequest

```rust
pub struct OriginRequest {
    pub method: String,
    pub target: String,
    pub headers: Vec<OriginHeader>,
    pub client_address: Option<String>,
}
```

The head of one public request a relay `ingress` plugin carries
(`Origin.fetch`). Its scheme is `https` and its authority is the library's
public origin; neither is the plugin's to choose.

**Fields**

- `method`: Request method, an HTTP token (`GET`, `PUT`, ...).
- `target`: Path and query in origin form (`/api/status?x=1`), at most
  `maxOriginTargetBytes`.
- `headers`: Request header fields in order, at most `maxOriginHeaders` and
  `maxOriginHeaderBytes`. `host` and forwarding headers are ignored.
- `client_address`: IP address of the client, from the provider's own connection metadata;
  empty when the provider gives none.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OriginRefusal

```rust
pub struct OriginRefusal {
    pub status: u16,
    pub headers: Vec<OriginHeader>,
    pub body: Vec<u8>,
}
```

The library's answer to an `Origin.connect` it did not upgrade: an
ordinary response (`401`, `403`, `404`, ...) for the plugin to hand back
to the client.

**Fields**

- `status`: HTTP status code, 200 to 599 and never `101`.
- `headers`: Response header fields in order, at most `maxOriginHeaders` and
  `maxOriginHeaderBytes`.
- `body`: Response body, at most `maxOriginRefusalBodyBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### IngressStarted

```rust
pub struct IngressStarted {
    pub origin: String,
}
```

Success payload of `Ingress.start`.

**Fields**

- `origin`: The public origin now serving: `https://` and a host name, no port, path
  or trailing slash, at most `maxIngressOriginBytes` after the scheme.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookUrl

```rust
pub struct WebhookUrl {
    pub url: Option<String>,
}
```

Success payload of `Webhooks.url`.

**Fields**

- `url`: Absolute `https://` URL; empty when the library cannot be reached from
  the Internet or webhooks are turned off, and the plugin should use its
  fallback.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookPullRequest

```rust
pub struct WebhookPullRequest {
    pub name: String,
    pub batch_size: u32,
    pub visibility_timeout_ms: u32,
}
```

Arguments of `Webhooks.pull`.

**Fields**

- `name`: `\[\[webhooks\]\]` name.
- `batch_size`: Most deliveries to lease, 1 to `maxWebhookPullBatch`.
- `visibility_timeout_ms`: How long leased deliveries stay hidden from the next pull, in
  milliseconds, at most `maxWebhookVisibilityMs`; zero means one minute.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookHeader

```rust
pub struct WebhookHeader {
    pub name: String,
    pub value: String,
}
```

One request header kept with a delivery.

**Fields**

- `name`: Lowercase header name, one the row's `headers` names or `content-type`.
- `value`: Header value as received.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookDelivery

```rust
pub struct WebhookDelivery {
    pub lease_id: String,
    pub received_at_unix_ms: u64,
    pub attempt: u32,
    pub headers: Vec<WebhookHeader>,
    pub body: Vec<u8>,
}
```

One stored request, leased to the guest.

**Fields**

- `lease_id`: Lease id for `ack`, at most `maxWebhookLeaseIdBytes`; a new one each
  time the delivery is pulled.
- `received_at_unix_ms`: When the host stored the request, in Unix milliseconds.
- `attempt`: How many times this delivery has been pulled, counting this one.
- `headers`: Kept request headers, in the order the row names them.
- `body`: Request body as received, at most the row's `max_body_bytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookPullPage

```rust
pub struct WebhookPullPage {
    pub deliveries: Vec<WebhookDelivery>,
    pub missed: bool,
}
```

Success payload of `Webhooks.pull`.

**Fields**

- `deliveries`: Leased deliveries, oldest first.
- `missed`: True when the host dropped deliveries for this webhook since the last
  pull (the waiting cap was full or a delivery expired unacknowledged):
  the guest should treat what it follows as incomplete.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### IngressHealth

```rust
pub struct IngressHealth {
    pub serving: bool,
    pub detail: Option<String>,
}
```

Success payload of `Ingress.health`.

**Fields**

- `serving`: True when the address is serving.
- `detail`: Why it is not serving, for the operator; empty when serving.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookAckRequest

```rust
pub struct WebhookAckRequest {
    pub name: String,
    pub acks: Vec<String>,
    pub retries: Vec<String>,
}
```

Arguments of `Webhooks.ack`.

**Fields**

- `name`: `\[\[webhooks\]\]` name.
- `acks`: Leases whose deliveries are done and are deleted.
- `retries`: Leases whose deliveries the next pull offers again.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### WebhookDelivered

```rust
pub struct WebhookDelivered {
    pub name: String,
    pub waiting: u32,
}
```

Argument of `WebhookHandler.delivered`.

**Fields**

- `name`: `\[\[webhooks\]\]` name.
- `waiting`: Deliveries waiting for this webhook when the host called.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ProvisionGrant

```rust
pub struct ProvisionGrant {
    pub kind: ProvisionGrantKind,
    pub token: Option<String>,
    pub expires_at_unix_ms: Option<u64>,
}
```

A short-lived credential for one guided setup run. The host never stores
or logs it; the plugin uses it only during the call.

**Fields**

- `kind`: Kind of grant.
- `token`: The token; empty for `none`. At most `maxProvisionTokenBytes`.
  `None` when absent (wire zero value).
- `expires_at_unix_ms`: When the token expires (UTC Unix milliseconds); zero when unknown or it
  does not expire.
  `None` when absent (wire zero value).

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ProvisionRequest

```rust
pub struct ProvisionRequest {
    pub grant: ProvisionGrant,
    pub inputs_json: String,
}
```

Arguments of `Provisioning.provision`.

**Fields**

- `grant`: The operator's grant; never `none`.
- `inputs_json`: The operator's choices as a JSON object keyed by `\[provisioning\] inputs`
  setting keys (`{"bucket": "books"}`); `{}` when none were asked for.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### ProvisionedSecret

```rust
pub struct ProvisionedSecret {
    pub key: String,
    pub value: String,
}
```

One credential a guided setup returns for the host to seal.

**Fields**

- `key`: A `secret` setting key listed in `\[provisioning\] outputs`.
- `value`: The credential, at most `maxProvisionSecretBytes`.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Provisioned

```rust
pub struct Provisioned {
    pub settings_json: String,
    pub secrets: Vec<ProvisionedSecret>,
    pub summary: Vec<String>,
}
```

Success payload of `Provisioning.provision`.

**Fields**

- `settings_json`: Non-secret settings to store, as a JSON object keyed by setting keys
  listed in `\[provisioning\] outputs`; at most `maxProvisionSettingsBytes`.
- `secrets`: Credentials to seal; each key at most once.
- `summary`: Short lines for the setup screen ("Created bucket books"), at most
  `maxProvisionSummaryLines` of `maxProvisionSummaryLineBytes`. Never a
  credential.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### DeprovisionRequest

```rust
pub struct DeprovisionRequest {
    pub grant: ProvisionGrant,
}
```

Arguments of `Provisioning.deprovision`.

**Fields**

- `grant`: The operator's grant, or kind `none` to use the stored secrets.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventPollParams

```rust
pub struct EventPollParams {
    pub ack_follows: bool,
}
```

Params of `RemoteLibrary.pollEvents`.

**Fields**

- `ack_follows`: The host calls `ackEvents` with every user it passes on, so the guest
  keeps returning a user until it is acknowledged. False from hosts that
  predate `ackEvents`: the guest then treats a returned user as delivered.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### EventAckParams

```rust
pub struct EventAckParams {
    pub external_user_ids: Vec<String>,
}
```

Params of `RemoteLibrary.ackEvents`.

**Fields**

- `external_user_ids`: `ExternalUser.externalUserId` of each user the host passed on.

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
    Metadata,
    Covers,
    MetadataFiles,
    Notifier,
    Ingress,
    Ai,
    Provisioning,
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
- `Metadata`: Metadata provider: `lookup`, `search` and `chapters`
  (`MetadataProvider`).
- `Covers`: Cover fetcher: `fetchCovers` (`CoverFetcher`).
- `MetadataFiles`: Metadata file dialects: `parse` and `render` (`MetadataFiles`).
- `Notifier`: Notification delivery: `send` and `validateDestination` (`Notifier`).
- `Ingress`: Public address provider: `start`, `health` and `stop` (`Ingress`).
- `Ai`: Model serving: `models` and `run` (`AiModels`).
- `Provisioning`: Guided setup: `provision` and `deprovision` (`Provisioning`).

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

### WorkKind

```rust
pub enum WorkKind {
    Unspecified,
    Book,
    Podcast,
}
```

What family of creative work a record describes (FRBR work). Every record
names one: `unspecified` is refused by the host, so a guest that forgets
the field fails instead of silently describing a book. Whether a podcast
work is a show or an episode is structural, not a kind: an episode names
its show through `parentProductId`, a show names no parent.

**Variants**

- `Unspecified`: Not set; the host refuses the record.
- `Book`: A book: fiction or non-fiction, in any expression (read, narrated,
  dramatised, illustrated) and on any carrier.
- `Podcast`: A podcast: the show, or one episode of it when the record names a
  parent.

#### WorkKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### WorkKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<WorkKind>`, `Hash`, `PartialEq`, `Serialize`

### ExpressionKind

```rust
pub enum ExpressionKind {
    Unspecified,
    Text,
    Narration,
    Dramatisation,
    Graphic,
    OriginalAudio,
}
```

How a work is realised (FRBR expression): the kind of content you get,
independent of the carrier. A translation is not a kind: the expression's
language and its translator credits say it. `unspecified` is refused.

**Variants**

- `Unspecified`: Not set; the host refuses the record.
- `Text`: The written text, as sold in print and ebook editions.
- `Narration`: A reading of the text by one or more narrators.
- `Dramatisation`: A full-cast dramatisation or performance of the work.
- `Graphic`: Sequential art: comics, graphic novels, manga.
- `OriginalAudio`: Audio that is itself the first realisation rather than a reading of a
  text: podcast episodes, lectures, single-voice originals.

#### ExpressionKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### ExpressionKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<ExpressionKind>`, `Hash`, `PartialEq`, `Serialize`

### MediaKind

```rust
pub enum MediaKind {
    Unspecified,
    Audio,
    Ebook,
    Print,
}
```

Carrier of an edition (FRBR manifestation): what form you receive.
Whether the host acquires an edition depends on the storefront's
`MediaSupport.fetches` and what the host supports, not on this value
alone. `unspecified` is refused.

**Variants**

- `Unspecified`: Not set; the host refuses the record.
- `Audio`: Downloadable audio: audiobooks and podcast episodes.
- `Ebook`: A downloadable document: EPUB, PDF, CBZ and similar.
- `Print`: A printed edition.

#### MediaKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### MediaKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<MediaKind>`, `Hash`, `PartialEq`, `Serialize`

### IdentifierScheme

```rust
pub enum IdentifierScheme {
    Isbn,
    Asin,
    OpenLibraryEdition,
    OpenLibraryWork,
}
```

Scheme of an external identifier. The scheme fixes whether the id names a
work or an edition, and whether it is strong (unique) or weak (shared).

**Variants**

- `Isbn`: ISBN-10 or ISBN-13 of an edition; weak, because one ISBN can sit on
  several editions. The host normalises it to ISBN-13.
- `Asin`: Amazon ASIN of an edition in one marketplace; strong.
- `OpenLibraryEdition`: OpenLibrary edition id (`OL…M`); strong.
- `OpenLibraryWork`: OpenLibrary work id (`OL…W`); names the work, not an edition.

#### IdentifierScheme::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### IdentifierScheme::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<IdentifierScheme>`, `Hash`, `PartialEq`, `Serialize`

### CreditRole

```rust
pub enum CreditRole {
    Author,
    Narrator,
    Translator,
    Editor,
    Illustrator,
    Publisher,
    Host,
    Cast,
    Adaptor,
}
```

A contributor's responsibility for a work, expression or edition.

**Variants**

- `Author`: Wrote the work.
- `Narrator`: Reads the text in a narration.
- `Translator`: Translated the text.
- `Editor`: Edited the work or the edition.
- `Illustrator`: Illustrated the work or the edition.
- `Publisher`: Published the edition.
- `Host`: Hosts a podcast show or episode.
- `Cast`: Performs a part in a dramatisation.
- `Adaptor`: Adapted the work into a dramatisation.

#### CreditRole::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### CreditRole::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CreditRole>`, `Hash`, `PartialEq`, `Serialize`

### CreditLevel

```rust
pub enum CreditLevel {
    RoleDefault,
    Work,
    Expression,
    Edition,
}
```

Which level a credit attaches to. Ordinal 0 is the role's usual level, so
a guest sets this only when a credit departs from it.

**Variants**

- `RoleDefault`: The role's usual level: author and adaptor on the work; narrator,
  translator, host and cast on the expression; editor, illustrator and
  publisher on the edition.
- `Work`: The work.
- `Expression`: The expression.
- `Edition`: The edition.

#### CreditLevel::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### CreditLevel::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<CreditLevel>`, `Hash`, `PartialEq`, `Serialize`

### FetchedFileRole

```rust
pub enum FetchedFileRole {
    Unspecified,
    Part,
    Whole,
    Cover,
}
```

What one fetched file is to the edition it belongs to.

**Variants**

- `Unspecified`: Not set; the host refuses a file without a role.
- `Part`: One ordered piece of the edition (an audio part or chapter file, one
  volume of a set). The host joins parts in list order.
- `Whole`: The whole edition in one file (an assembled M4B, an EPUB, a PDF).
- `Cover`: The edition's cover image.

#### FetchedFileRole::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### FetchedFileRole::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<FetchedFileRole>`, `Hash`, `PartialEq`, `Serialize`

### ChapterKind

```rust
pub enum ChapterKind {
    Unknown,
    Chapters,
    Tracks,
}
```

What a storefront's chapter markers mark (`PlainFetch.chapterKind`).

**Variants**

- `Unknown`: The storefront does not say.
- `Chapters`: Literary chapters: what a listener would call the book's chapters.
- `Tracks`: Track or part boundaries: where the store cut its files, which need
  not fall on a chapter.

#### ChapterKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### ChapterKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<ChapterKind>`, `Hash`, `PartialEq`, `Serialize`

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

### PriceLookup

```rust
pub enum PriceLookup {
    Unspecified,
    Product,
    Isbn,
    Asin,
    Title,
}
```

How a purchase-hint query names the title to price.

**Variants**

- `Unspecified`: Any field may be used, strongest first (older hosts).
- `Product`: `productId` names the store's own product; answer for it alone.
- `Isbn`: `isbn` names the edition; answer for that exact ISBN or not found.
  Sent only to a store that declared `CatalogCapabilities.priceByIsbn`.
- `Asin`: `asin` names the edition; answer for that exact ASIN or not found.
  Sent only to a store that declared `CatalogCapabilities.priceByAsin`.
- `Title`: Search by `title` and `authors`, and return every priced listing the
  search matched in `PurchaseHintResult.records`, so the host can place
  each one in the library model and keep only the recording it asked for.

#### PriceLookup::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### PriceLookup::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<PriceLookup>`, `Hash`, `PartialEq`, `Serialize`

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

### Explicitness

```rust
pub enum Explicitness {
    Unknown,
    Clean,
    Explicit,
}
```

Whether an edition is marked as explicit (adult) content. A patron scope
can hide explicit editions; `unknown` and `clean` editions stay visible.

**Variants**

- `Unknown`: The storefront or file did not say.
- `Clean`: Marked as not explicit.
- `Explicit`: Marked as explicit (adult) content.

#### Explicitness::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### Explicitness::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<Explicitness>`, `Hash`, `PartialEq`, `Serialize`

### OfferedCodec

```rust
pub enum OfferedCodec {
    Unknown,
    Aac,
    Alac,
    Flac,
    Mp3,
    Opus,
    Vorbis,
    Eac3,
    Ac4,
}
```

Audio codec of a stream a store offers (`QualityOffer.codec`). Spelled as
the host's probed codecs are, so an offer compares with a held file.

**Variants**

- `Unknown`: The store did not say; the host never upgrades to an unknown codec.
- `Aac`: MPEG-4 AAC, lossy.
- `Alac`: Apple Lossless.
- `Flac`: FLAC, lossless.
- `Mp3`: MPEG audio layer III (or I / II), lossy.
- `Opus`: Opus, lossy.
- `Vorbis`: Vorbis, lossy.
- `Eac3`: Dolby Digital Plus (E-AC-3), lossy.
- `Ac4`: Dolby AC-4, lossy.

#### OfferedCodec::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### OfferedCodec::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<OfferedCodec>`, `Hash`, `PartialEq`, `Serialize`

### OfferedAacProfile

```rust
pub enum OfferedAacProfile {
    Unknown,
    Lc,
    He,
    HeV2,
    Xhe,
}
```

AAC profile of an offered AAC stream (`QualityOffer.aacProfile`).

**Variants**

- `Unknown`: Not AAC, or the store did not say.
- `Lc`: AAC-LC.
- `He`: HE-AAC (AAC-LC plus SBR).
- `HeV2`: HE-AAC v2 (SBR plus parametric stereo).
- `Xhe`: xHE-AAC (USAC).

#### OfferedAacProfile::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### OfferedAacProfile::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<OfferedAacProfile>`, `Hash`, `PartialEq`, `Serialize`

### InvocationPurpose

```rust
pub enum InvocationPurpose {
    Serve,
    Provision,
}
```

Why the host opens a plugin's entrypoints (`Invocation.purpose`).

**Variants**

- `Serve`: Ordinary work: every required setting is set.
- `Provision`: Guided setup (`Entrypoints.provisioning`): settings may be incomplete.

#### InvocationPurpose::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### InvocationPurpose::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<InvocationPurpose>`, `Hash`, `PartialEq`, `Serialize`

### RemoteMedia

```rust
pub enum RemoteMedia {
    Image,
    Audio,
}
```

The kind of public media `CoverFetcher.fetchCovers` downloads.

**Variants**

- `Image`: Cover art or another image.
- `Audio`: A short audio sample of a book.

#### RemoteMedia::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### RemoteMedia::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<RemoteMedia>`, `Hash`, `PartialEq`, `Serialize`

### MetadataField

```rust
pub enum MetadataField {
    Title,
    Subtitle,
    Description,
    Publisher,
    PublishedAt,
    Language,
    Genres,
    Tags,
    Abridgement,
    ExpressionKind,
    Identifiers,
    Credits,
    Series,
    Chapters,
    Duration,
    Explicitness,
}
```

A field of a `MetadataFileRecord`. Dialects declare in the manifest which
fields their format carries; a parsed record lists the ones its file
states, and a render request lists the ones to write.

**Variants**

- `Title`: `MetadataFileRecord.title`.
- `Subtitle`: `MetadataFileRecord.subtitle`.
- `Description`: `MetadataFileRecord.description`.
- `Publisher`: `MetadataFileRecord.publisher`.
- `PublishedAt`: `MetadataFileRecord.publishedAt`.
- `Language`: `MetadataFileRecord.language`.
- `Genres`: `MetadataFileRecord.genres`.
- `Tags`: `MetadataFileRecord.tags`.
- `Abridgement`: `MetadataFileRecord.abridgement`.
- `ExpressionKind`: `MetadataFileRecord.expressionKind`.
- `Identifiers`: `MetadataFileRecord.identifiers`.
- `Credits`: `MetadataFileRecord.credits` in the roles `creditRoles` names.
- `Series`: `MetadataFileRecord.seriesEntries`.
- `Chapters`: `MetadataFileRecord.chapters`.
- `Duration`: `MetadataFileRecord.durationMs`.
- `Explicitness`: `MetadataFileRecord.explicitness`.

#### MetadataField::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### MetadataField::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<MetadataField>`, `Hash`, `PartialEq`, `Serialize`

### NotificationSeverity

```rust
pub enum NotificationSeverity {
    Info,
    Warning,
    Critical,
}
```

How urgent a notification is. Routing rules carry a minimum severity.

**Variants**

- `Info`: Informational.
- `Warning`: Needs attention soon.
- `Critical`: Needs attention now; ignores quiet hours for administrators.

#### NotificationSeverity::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### NotificationSeverity::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<NotificationSeverity>`, `Hash`, `PartialEq`, `Serialize`

### NotificationAudience

```rust
pub enum NotificationAudience {
    Operational,
    Patron,
    Account,
}
```

Who a notification type is for. The account audience is reserved to the
host; a plugin may declare only operational and patron types.

**Variants**

- `Operational`: Administrators and library channels.
- `Patron`: One patron, under their own preferences.
- `Account`: Account mail and security notices; host-only.

#### NotificationAudience::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### NotificationAudience::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<NotificationAudience>`, `Hash`, `PartialEq`, `Serialize`

### NotificationAlertAction

```rust
pub enum NotificationAlertAction {
    None,
    Trigger,
    Resolve,
}
```

Lifecycle step of an alert, for services such as PagerDuty that open and
close incidents.

**Variants**

- `None`: Not an alert: a one-off message.
- `Trigger`: Open (or update) the alert named by `alertKey`.
- `Resolve`: Close the alert named by `alertKey`.

#### NotificationAlertAction::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### NotificationAlertAction::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<NotificationAlertAction>`, `Hash`, `PartialEq`, `Serialize`

### NotifyDestinationKind

```rust
pub enum NotifyDestinationKind {
    Channel,
    Personal,
}
```

Kind of destination a `Notifier` sends to.

**Variants**

- `Channel`: A library channel the instance itself is configured for (its settings
  hold the webhook or service key); `addressJson` is empty.
- `Personal`: One patron's own address on this instance (`addressJson`).

#### NotifyDestinationKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### NotifyDestinationKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<NotifyDestinationKind>`, `Hash`, `PartialEq`, `Serialize`

### AiTask

```rust
pub enum AiTask {
    TextEmbeddings,
}
```

What a model does. A model has exactly one task, and the task fixes the
shape of `AiRunParams.input` and of the reply.

**Variants**

- `TextEmbeddings`: Turns each text into one vector; similar texts get nearby vectors.

#### AiTask::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### AiTask::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<AiTask>`, `Hash`, `PartialEq`, `Serialize`

### AiTextRole

```rust
pub enum AiTextRole {
    Document,
    Query,
}
```

What a text is to an embedding model. Some models embed a search query
differently from the documents it is matched against.

**Variants**

- `Document`: Stored content: a description, a passage.
- `Query`: A search query matched against documents.

#### AiTextRole::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### AiTextRole::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<AiTextRole>`, `Hash`, `PartialEq`, `Serialize`

### ProvisionGrantKind

```rust
pub enum ProvisionGrantKind {
    None,
    ApiToken,
    OauthAccessToken,
}
```

What kind of grant the operator gave a guided setup (`ProvisionGrant`).

**Variants**

- `None`: No grant: `deprovision` uses the instance's stored secrets.
- `ApiToken`: An API token the operator created at the provider and pasted.
- `OauthAccessToken`: An OAuth access token from the provider's sign-in.

#### ProvisionGrantKind::wire_name

```rust
pub fn wire_name(self) -> &'static str
```

Wire name of this enumerant (matches the TypeScript / Python SDK literal).

#### ProvisionGrantKind::from_wire_name

```rust
pub fn from_wire_name(name: &str) -> Option<Self>
```

Parse a wire name; `None` for unknown strings.

Implements: `Clone`, `Copy`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `From<ProvisionGrantKind>`, `Hash`, `PartialEq`, `Serialize`

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

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### SocketFrame

```rust
pub enum SocketFrame {
    Text(String),
    Binary(Vec<u8>),
}
```

One WebSocket message, or one chunk of a `tls://` stream (always
`binary`).

**Variants**

- `Text`: Text message, UTF-8.
- `Binary`: Binary message.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### OriginConnected

```rust
pub enum OriginConnected {
    Socket(SocketInfo),
    Refused(OriginRefusal),
}
```

Success payload of `Origin.connect`.

**Variants**

- `Socket`: The library accepted the upgrade; the socket is open in
  `Bindings.sockets`. Its `protocol` is the subprotocol the library
  chose.
- `Refused`: The library answered without upgrading.

Implements: `Clone`, `Debug`, `Default`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

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

### write_catalog_capabilities

```rust
pub fn write_catalog_capabilities(b: Builder<'_>, v: &CatalogCapabilities) -> Result<()>
```

Encode a [`CatalogCapabilities`](#catalogcapabilities) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_catalog_capabilities

```rust
pub fn read_catalog_capabilities(r: Reader<'_>) -> Result<CatalogCapabilities>
```

Decode a [`CatalogCapabilities`](#catalogcapabilities) from a Cap'n Proto reader.

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

### write_media_support

```rust
pub fn write_media_support(b: Builder<'_>, v: &MediaSupport) -> Result<()>
```

Encode a [`MediaSupport`](#mediasupport) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_media_support

```rust
pub fn read_media_support(r: Reader<'_>) -> Result<MediaSupport>
```

Decode a [`MediaSupport`](#mediasupport) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_identifier

```rust
pub fn write_identifier(b: Builder<'_>, v: &Identifier) -> Result<()>
```

Encode a [`Identifier`](#identifier) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_identifier

```rust
pub fn read_identifier(r: Reader<'_>) -> Result<Identifier>
```

Decode a [`Identifier`](#identifier) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_credit

```rust
pub fn write_credit(b: Builder<'_>, v: &Credit) -> Result<()>
```

Encode a [`Credit`](#credit) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_credit

```rust
pub fn read_credit(r: Reader<'_>) -> Result<Credit>
```

Decode a [`Credit`](#credit) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_series_entry

```rust
pub fn write_series_entry(b: Builder<'_>, v: &SeriesEntry) -> Result<()>
```

Encode a [`SeriesEntry`](#seriesentry) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_series_entry

```rust
pub fn read_series_entry(r: Reader<'_>) -> Result<SeriesEntry>
```

Decode a [`SeriesEntry`](#seriesentry) from a Cap'n Proto reader.

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

### write_scan_page

```rust
pub fn write_scan_page(b: Builder<'_>, v: &ScanPage) -> Result<()>
```

Encode a [`ScanPage`](#scanpage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scan_page

```rust
pub fn read_scan_page(r: Reader<'_>) -> Result<ScanPage>
```

Decode a [`ScanPage`](#scanpage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_scan_reply

```rust
pub fn write_scan_reply(b: Builder<'_>, v: &Result<ScanPage, PluginError>) -> Result<()>
```

Encode a `ScanReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_scan_reply

```rust
pub fn read_scan_reply(r: Reader<'_>) -> Result<Result<ScanPage, PluginError>>
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

### write_fetched_file

```rust
pub fn write_fetched_file(b: Builder<'_>, v: &FetchedFile) -> Result<()>
```

Encode a [`FetchedFile`](#fetchedfile) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_fetched_file

```rust
pub fn read_fetched_file(r: Reader<'_>) -> Result<FetchedFile>
```

Decode a [`FetchedFile`](#fetchedfile) from a Cap'n Proto reader.

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

### write_revoke_account_params

```rust
pub fn write_revoke_account_params(b: Builder<'_>, v: &RevokeAccountParams) -> Result<()>
```

Encode a [`RevokeAccountParams`](#revokeaccountparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_revoke_account_params

```rust
pub fn read_revoke_account_params(r: Reader<'_>) -> Result<RevokeAccountParams>
```

Decode a [`RevokeAccountParams`](#revokeaccountparams) from a Cap'n Proto reader.

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

### write_priced_record

```rust
pub fn write_priced_record(b: Builder<'_>, v: &PricedRecord) -> Result<()>
```

Encode a [`PricedRecord`](#pricedrecord) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_priced_record

```rust
pub fn read_priced_record(r: Reader<'_>) -> Result<PricedRecord>
```

Decode a [`PricedRecord`](#pricedrecord) from a Cap'n Proto reader.

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

### write_purchase_hints_params

```rust
pub fn write_purchase_hints_params(b: Builder<'_>, v: &PurchaseHintsParams) -> Result<()>
```

Encode a [`PurchaseHintsParams`](#purchasehintsparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_purchase_hints_params

```rust
pub fn read_purchase_hints_params(r: Reader<'_>) -> Result<PurchaseHintsParams>
```

Decode a [`PurchaseHintsParams`](#purchasehintsparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_purchase_hints_result

```rust
pub fn write_purchase_hints_result(b: Builder<'_>, v: &PurchaseHintsResult) -> Result<()>
```

Encode a [`PurchaseHintsResult`](#purchasehintsresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_purchase_hints_result

```rust
pub fn read_purchase_hints_result(r: Reader<'_>) -> Result<PurchaseHintsResult>
```

Decode a [`PurchaseHintsResult`](#purchasehintsresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_purchase_hints_reply

```rust
pub fn write_purchase_hints_reply(
    b: Builder<'_>,
    v: &Result<PurchaseHintsResult, PluginError>,
) -> Result<()>
```

Encode a `PurchaseHintsReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_purchase_hints_reply

```rust
pub fn read_purchase_hints_reply(
    r: Reader<'_>,
) -> Result<Result<PurchaseHintsResult, PluginError>>
```

Decode a `PurchaseHintsReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_quality_offer

```rust
pub fn write_quality_offer(b: Builder<'_>, v: &QualityOffer) -> Result<()>
```

Encode a [`QualityOffer`](#qualityoffer) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_quality_offer

```rust
pub fn read_quality_offer(r: Reader<'_>) -> Result<QualityOffer>
```

Decode a [`QualityOffer`](#qualityoffer) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_quality_offers_params

```rust
pub fn write_quality_offers_params(b: Builder<'_>, v: &QualityOffersParams) -> Result<()>
```

Encode a [`QualityOffersParams`](#qualityoffersparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_quality_offers_params

```rust
pub fn read_quality_offers_params(r: Reader<'_>) -> Result<QualityOffersParams>
```

Decode a [`QualityOffersParams`](#qualityoffersparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_title_quality_offer

```rust
pub fn write_title_quality_offer(b: Builder<'_>, v: &TitleQualityOffer) -> Result<()>
```

Encode a [`TitleQualityOffer`](#titlequalityoffer) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_title_quality_offer

```rust
pub fn read_title_quality_offer(r: Reader<'_>) -> Result<TitleQualityOffer>
```

Decode a [`TitleQualityOffer`](#titlequalityoffer) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_quality_offers_result

```rust
pub fn write_quality_offers_result(b: Builder<'_>, v: &QualityOffersResult) -> Result<()>
```

Encode a [`QualityOffersResult`](#qualityoffersresult) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_quality_offers_result

```rust
pub fn read_quality_offers_result(r: Reader<'_>) -> Result<QualityOffersResult>
```

Decode a [`QualityOffersResult`](#qualityoffersresult) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_quality_offers_reply

```rust
pub fn write_quality_offers_reply(
    b: Builder<'_>,
    v: &Result<QualityOffersResult, PluginError>,
) -> Result<()>
```

Encode a `QualityOffersReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_quality_offers_reply

```rust
pub fn read_quality_offers_reply(
    r: Reader<'_>,
) -> Result<Result<QualityOffersResult, PluginError>>
```

Decode a `QualityOffersReply` union into a typed result.

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

### write_scheduled_controller

```rust
pub fn write_scheduled_controller(b: Builder<'_>, v: &ScheduledController) -> Result<()>
```

Encode a [`ScheduledController`](#scheduledcontroller) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_scheduled_controller

```rust
pub fn read_scheduled_controller(r: Reader<'_>) -> Result<ScheduledController>
```

Decode a [`ScheduledController`](#scheduledcontroller) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_lookup_params

```rust
pub fn write_metadata_lookup_params(
    b: Builder<'_>,
    v: &MetadataLookupParams,
) -> Result<()>
```

Encode a [`MetadataLookupParams`](#metadatalookupparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_lookup_params

```rust
pub fn read_metadata_lookup_params(r: Reader<'_>) -> Result<MetadataLookupParams>
```

Decode a [`MetadataLookupParams`](#metadatalookupparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_search_params

```rust
pub fn write_metadata_search_params(
    b: Builder<'_>,
    v: &MetadataSearchParams,
) -> Result<()>
```

Encode a [`MetadataSearchParams`](#metadatasearchparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_search_params

```rust
pub fn read_metadata_search_params(r: Reader<'_>) -> Result<MetadataSearchParams>
```

Decode a [`MetadataSearchParams`](#metadatasearchparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_chapters_params

```rust
pub fn write_metadata_chapters_params(
    b: Builder<'_>,
    v: &MetadataChaptersParams,
) -> Result<()>
```

Encode a [`MetadataChaptersParams`](#metadatachaptersparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_chapters_params

```rust
pub fn read_metadata_chapters_params(r: Reader<'_>) -> Result<MetadataChaptersParams>
```

Decode a [`MetadataChaptersParams`](#metadatachaptersparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_record

```rust
pub fn write_metadata_record(b: Builder<'_>, v: &MetadataRecord) -> Result<()>
```

Encode a [`MetadataRecord`](#metadatarecord) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_record

```rust
pub fn read_metadata_record(r: Reader<'_>) -> Result<MetadataRecord>
```

Decode a [`MetadataRecord`](#metadatarecord) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_records

```rust
pub fn write_metadata_records(b: Builder<'_>, v: &MetadataRecords) -> Result<()>
```

Encode a [`MetadataRecords`](#metadatarecords) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_records

```rust
pub fn read_metadata_records(r: Reader<'_>) -> Result<MetadataRecords>
```

Decode a [`MetadataRecords`](#metadatarecords) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_records_reply

```rust
pub fn write_metadata_records_reply(
    b: Builder<'_>,
    v: &Result<MetadataRecords, PluginError>,
) -> Result<()>
```

Encode a `MetadataRecordsReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_metadata_records_reply

```rust
pub fn read_metadata_records_reply(
    r: Reader<'_>,
) -> Result<Result<MetadataRecords, PluginError>>
```

Decode a `MetadataRecordsReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_metadata_chapter

```rust
pub fn write_metadata_chapter(b: Builder<'_>, v: &MetadataChapter) -> Result<()>
```

Encode a [`MetadataChapter`](#metadatachapter) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_chapter

```rust
pub fn read_metadata_chapter(r: Reader<'_>) -> Result<MetadataChapter>
```

Decode a [`MetadataChapter`](#metadatachapter) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_chapters

```rust
pub fn write_metadata_chapters(b: Builder<'_>, v: &MetadataChapters) -> Result<()>
```

Encode a [`MetadataChapters`](#metadatachapters) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_chapters

```rust
pub fn read_metadata_chapters(r: Reader<'_>) -> Result<MetadataChapters>
```

Decode a [`MetadataChapters`](#metadatachapters) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_chapters_reply

```rust
pub fn write_metadata_chapters_reply(
    b: Builder<'_>,
    v: &Result<MetadataChapters, PluginError>,
) -> Result<()>
```

Encode a `MetadataChaptersReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_metadata_chapters_reply

```rust
pub fn read_metadata_chapters_reply(
    r: Reader<'_>,
) -> Result<Result<MetadataChapters, PluginError>>
```

Decode a `MetadataChaptersReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_cover_fetch_params

```rust
pub fn write_cover_fetch_params(b: Builder<'_>, v: &CoverFetchParams) -> Result<()>
```

Encode a [`CoverFetchParams`](#coverfetchparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cover_fetch_params

```rust
pub fn read_cover_fetch_params(r: Reader<'_>) -> Result<CoverFetchParams>
```

Decode a [`CoverFetchParams`](#coverfetchparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cover_condition

```rust
pub fn write_cover_condition(b: Builder<'_>, v: &CoverCondition) -> Result<()>
```

Encode a [`CoverCondition`](#covercondition) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cover_condition

```rust
pub fn read_cover_condition(r: Reader<'_>) -> Result<CoverCondition>
```

Decode a [`CoverCondition`](#covercondition) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cover_image

```rust
pub fn write_cover_image(b: Builder<'_>, v: &CoverImage) -> Result<()>
```

Encode a [`CoverImage`](#coverimage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cover_image

```rust
pub fn read_cover_image(r: Reader<'_>) -> Result<CoverImage>
```

Decode a [`CoverImage`](#coverimage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cover_images

```rust
pub fn write_cover_images(b: Builder<'_>, v: &CoverImages) -> Result<()>
```

Encode a [`CoverImages`](#coverimages) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_cover_images

```rust
pub fn read_cover_images(r: Reader<'_>) -> Result<CoverImages>
```

Decode a [`CoverImages`](#coverimages) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_cover_images_reply

```rust
pub fn write_cover_images_reply(
    b: Builder<'_>,
    v: &Result<CoverImages, PluginError>,
) -> Result<()>
```

Encode a `CoverImagesReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_cover_images_reply

```rust
pub fn read_cover_images_reply(r: Reader<'_>) -> Result<Result<CoverImages, PluginError>>
```

Decode a `CoverImagesReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_metadata_file_record

```rust
pub fn write_metadata_file_record(b: Builder<'_>, v: &MetadataFileRecord) -> Result<()>
```

Encode a [`MetadataFileRecord`](#metadatafilerecord) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_file_record

```rust
pub fn read_metadata_file_record(r: Reader<'_>) -> Result<MetadataFileRecord>
```

Decode a [`MetadataFileRecord`](#metadatafilerecord) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_parse_params

```rust
pub fn write_metadata_file_parse_params(
    b: Builder<'_>,
    v: &MetadataFileParseParams,
) -> Result<()>
```

Encode a [`MetadataFileParseParams`](#metadatafileparseparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_file_parse_params

```rust
pub fn read_metadata_file_parse_params(r: Reader<'_>) -> Result<MetadataFileParseParams>
```

Decode a [`MetadataFileParseParams`](#metadatafileparseparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_parsed

```rust
pub fn write_metadata_file_parsed(b: Builder<'_>, v: &MetadataFileParsed) -> Result<()>
```

Encode a [`MetadataFileParsed`](#metadatafileparsed) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_file_parsed

```rust
pub fn read_metadata_file_parsed(r: Reader<'_>) -> Result<MetadataFileParsed>
```

Decode a [`MetadataFileParsed`](#metadatafileparsed) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_parse_reply

```rust
pub fn write_metadata_file_parse_reply(
    b: Builder<'_>,
    v: &Result<MetadataFileParsed, PluginError>,
) -> Result<()>
```

Encode a `MetadataFileParseReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_metadata_file_parse_reply

```rust
pub fn read_metadata_file_parse_reply(
    r: Reader<'_>,
) -> Result<Result<MetadataFileParsed, PluginError>>
```

Decode a `MetadataFileParseReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_metadata_audio_file

```rust
pub fn write_metadata_audio_file(b: Builder<'_>, v: &MetadataAudioFile) -> Result<()>
```

Encode a [`MetadataAudioFile`](#metadataaudiofile) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_audio_file

```rust
pub fn read_metadata_audio_file(r: Reader<'_>) -> Result<MetadataAudioFile>
```

Decode a [`MetadataAudioFile`](#metadataaudiofile) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_render_params

```rust
pub fn write_metadata_file_render_params(
    b: Builder<'_>,
    v: &MetadataFileRenderParams,
) -> Result<()>
```

Encode a [`MetadataFileRenderParams`](#metadatafilerenderparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_file_render_params

```rust
pub fn read_metadata_file_render_params(r: Reader<'_>) -> Result<MetadataFileRenderParams>
```

Decode a [`MetadataFileRenderParams`](#metadatafilerenderparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_rendered

```rust
pub fn write_metadata_file_rendered(
    b: Builder<'_>,
    v: &MetadataFileRendered,
) -> Result<()>
```

Encode a [`MetadataFileRendered`](#metadatafilerendered) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_metadata_file_rendered

```rust
pub fn read_metadata_file_rendered(r: Reader<'_>) -> Result<MetadataFileRendered>
```

Decode a [`MetadataFileRendered`](#metadatafilerendered) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_metadata_file_render_reply

```rust
pub fn write_metadata_file_render_reply(
    b: Builder<'_>,
    v: &Result<MetadataFileRendered, PluginError>,
) -> Result<()>
```

Encode a `MetadataFileRenderReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_metadata_file_render_reply

```rust
pub fn read_metadata_file_render_reply(
    r: Reader<'_>,
) -> Result<Result<MetadataFileRendered, PluginError>>
```

Decode a `MetadataFileRenderReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_notification_field

```rust
pub fn write_notification_field(b: Builder<'_>, v: &NotificationField) -> Result<()>
```

Encode a [`NotificationField`](#notificationfield) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notification_field

```rust
pub fn read_notification_field(r: Reader<'_>) -> Result<NotificationField>
```

Decode a [`NotificationField`](#notificationfield) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notification_message

```rust
pub fn write_notification_message(b: Builder<'_>, v: &NotificationMessage) -> Result<()>
```

Encode a [`NotificationMessage`](#notificationmessage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notification_message

```rust
pub fn read_notification_message(r: Reader<'_>) -> Result<NotificationMessage>
```

Decode a [`NotificationMessage`](#notificationmessage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_destination

```rust
pub fn write_notify_destination(b: Builder<'_>, v: &NotifyDestination) -> Result<()>
```

Encode a [`NotifyDestination`](#notifydestination) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_destination

```rust
pub fn read_notify_destination(r: Reader<'_>) -> Result<NotifyDestination>
```

Decode a [`NotifyDestination`](#notifydestination) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_send_params

```rust
pub fn write_notify_send_params(b: Builder<'_>, v: &NotifySendParams) -> Result<()>
```

Encode a [`NotifySendParams`](#notifysendparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_send_params

```rust
pub fn read_notify_send_params(r: Reader<'_>) -> Result<NotifySendParams>
```

Decode a [`NotifySendParams`](#notifysendparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_sent

```rust
pub fn write_notify_sent(b: Builder<'_>, v: &NotifySent) -> Result<()>
```

Encode a [`NotifySent`](#notifysent) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_sent

```rust
pub fn read_notify_sent(r: Reader<'_>) -> Result<NotifySent>
```

Decode a [`NotifySent`](#notifysent) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_send_reply

```rust
pub fn write_notify_send_reply(
    b: Builder<'_>,
    v: &Result<NotifySent, PluginError>,
) -> Result<()>
```

Encode a `NotifySendReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_notify_send_reply

```rust
pub fn read_notify_send_reply(r: Reader<'_>) -> Result<Result<NotifySent, PluginError>>
```

Decode a `NotifySendReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_notify_validate_params

```rust
pub fn write_notify_validate_params(
    b: Builder<'_>,
    v: &NotifyValidateParams,
) -> Result<()>
```

Encode a [`NotifyValidateParams`](#notifyvalidateparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_validate_params

```rust
pub fn read_notify_validate_params(r: Reader<'_>) -> Result<NotifyValidateParams>
```

Decode a [`NotifyValidateParams`](#notifyvalidateparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_validated

```rust
pub fn write_notify_validated(b: Builder<'_>, v: &NotifyValidated) -> Result<()>
```

Encode a [`NotifyValidated`](#notifyvalidated) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_validated

```rust
pub fn read_notify_validated(r: Reader<'_>) -> Result<NotifyValidated>
```

Decode a [`NotifyValidated`](#notifyvalidated) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_validate_reply

```rust
pub fn write_notify_validate_reply(
    b: Builder<'_>,
    v: &Result<NotifyValidated, PluginError>,
) -> Result<()>
```

Encode a `NotifyValidateReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_notify_validate_reply

```rust
pub fn read_notify_validate_reply(
    r: Reader<'_>,
) -> Result<Result<NotifyValidated, PluginError>>
```

Decode a `NotifyValidateReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_notify_raise_params

```rust
pub fn write_notify_raise_params(b: Builder<'_>, v: &NotifyRaiseParams) -> Result<()>
```

Encode a [`NotifyRaiseParams`](#notifyraiseparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_raise_params

```rust
pub fn read_notify_raise_params(r: Reader<'_>) -> Result<NotifyRaiseParams>
```

Decode a [`NotifyRaiseParams`](#notifyraiseparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_raised

```rust
pub fn write_notify_raised(b: Builder<'_>, v: &NotifyRaised) -> Result<()>
```

Encode a [`NotifyRaised`](#notifyraised) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_notify_raised

```rust
pub fn read_notify_raised(r: Reader<'_>) -> Result<NotifyRaised>
```

Decode a [`NotifyRaised`](#notifyraised) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_notify_raise_reply

```rust
pub fn write_notify_raise_reply(
    b: Builder<'_>,
    v: &Result<NotifyRaised, PluginError>,
) -> Result<()>
```

Encode a `NotifyRaiseReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_notify_raise_reply

```rust
pub fn read_notify_raise_reply(r: Reader<'_>) -> Result<Result<NotifyRaised, PluginError>>
```

Decode a `NotifyRaiseReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_ai_model

```rust
pub fn write_ai_model(b: Builder<'_>, v: &AiModel) -> Result<()>
```

Encode a [`AiModel`](#aimodel) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_model

```rust
pub fn read_ai_model(r: Reader<'_>) -> Result<AiModel>
```

Decode a [`AiModel`](#aimodel) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_model_list

```rust
pub fn write_ai_model_list(b: Builder<'_>, v: &AiModelList) -> Result<()>
```

Encode a [`AiModelList`](#aimodellist) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_model_list

```rust
pub fn read_ai_model_list(r: Reader<'_>) -> Result<AiModelList>
```

Decode a [`AiModelList`](#aimodellist) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_models_reply

```rust
pub fn write_ai_models_reply(
    b: Builder<'_>,
    v: &Result<AiModelList, PluginError>,
) -> Result<()>
```

Encode a `AiModelsReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_ai_models_reply

```rust
pub fn read_ai_models_reply(r: Reader<'_>) -> Result<Result<AiModelList, PluginError>>
```

Decode a `AiModelsReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_ai_embed_text

```rust
pub fn write_ai_embed_text(b: Builder<'_>, v: &AiEmbedText) -> Result<()>
```

Encode a [`AiEmbedText`](#aiembedtext) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_embed_text

```rust
pub fn read_ai_embed_text(r: Reader<'_>) -> Result<AiEmbedText>
```

Decode a [`AiEmbedText`](#aiembedtext) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_text_embeddings_input

```rust
pub fn write_ai_text_embeddings_input(
    b: Builder<'_>,
    v: &AiTextEmbeddingsInput,
) -> Result<()>
```

Encode a [`AiTextEmbeddingsInput`](#aitextembeddingsinput) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_text_embeddings_input

```rust
pub fn read_ai_text_embeddings_input(r: Reader<'_>) -> Result<AiTextEmbeddingsInput>
```

Decode a [`AiTextEmbeddingsInput`](#aitextembeddingsinput) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_text_embeddings_output

```rust
pub fn write_ai_text_embeddings_output(
    b: Builder<'_>,
    v: &AiTextEmbeddingsOutput,
) -> Result<()>
```

Encode a [`AiTextEmbeddingsOutput`](#aitextembeddingsoutput) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_text_embeddings_output

```rust
pub fn read_ai_text_embeddings_output(r: Reader<'_>) -> Result<AiTextEmbeddingsOutput>
```

Decode a [`AiTextEmbeddingsOutput`](#aitextembeddingsoutput) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_input

```rust
pub fn write_ai_input(b: Builder<'_>, v: &AiInput) -> Result<()>
```

Encode a [`AiInput`](#aiinput) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_input

```rust
pub fn read_ai_input(r: Reader<'_>) -> Result<AiInput>
```

Decode a [`AiInput`](#aiinput) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_output

```rust
pub fn write_ai_output(b: Builder<'_>, v: &AiOutput) -> Result<()>
```

Encode a [`AiOutput`](#aioutput) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_output

```rust
pub fn read_ai_output(r: Reader<'_>) -> Result<AiOutput>
```

Decode a [`AiOutput`](#aioutput) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_run_params

```rust
pub fn write_ai_run_params(b: Builder<'_>, v: &AiRunParams) -> Result<()>
```

Encode a [`AiRunParams`](#airunparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_run_params

```rust
pub fn read_ai_run_params(r: Reader<'_>) -> Result<AiRunParams>
```

Decode a [`AiRunParams`](#airunparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_run_ok

```rust
pub fn write_ai_run_ok(b: Builder<'_>, v: &AiRunOk) -> Result<()>
```

Encode a [`AiRunOk`](#airunok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ai_run_ok

```rust
pub fn read_ai_run_ok(r: Reader<'_>) -> Result<AiRunOk>
```

Decode a [`AiRunOk`](#airunok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ai_run_reply

```rust
pub fn write_ai_run_reply(b: Builder<'_>, v: &Result<AiRunOk, PluginError>) -> Result<()>
```

Encode a `AiRunReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_ai_run_reply

```rust
pub fn read_ai_run_reply(r: Reader<'_>) -> Result<Result<AiRunOk, PluginError>>
```

Decode a `AiRunReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_socket_frame

```rust
pub fn write_socket_frame(b: Builder<'_>, v: &SocketFrame) -> Result<()>
```

Encode a `SocketFrame` union.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_socket_frame

```rust
pub fn read_socket_frame(r: Reader<'_>) -> Result<SocketFrame>
```

Decode a `SocketFrame` union.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_socket_header

```rust
pub fn write_socket_header(b: Builder<'_>, v: &SocketHeader) -> Result<()>
```

Encode a [`SocketHeader`](#socketheader) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_header

```rust
pub fn read_socket_header(r: Reader<'_>) -> Result<SocketHeader>
```

Decode a [`SocketHeader`](#socketheader) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_auto_response

```rust
pub fn write_socket_auto_response(b: Builder<'_>, v: &SocketAutoResponse) -> Result<()>
```

Encode a [`SocketAutoResponse`](#socketautoresponse) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_auto_response

```rust
pub fn read_socket_auto_response(r: Reader<'_>) -> Result<SocketAutoResponse>
```

Decode a [`SocketAutoResponse`](#socketautoresponse) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_connect_request

```rust
pub fn write_socket_connect_request(
    b: Builder<'_>,
    v: &SocketConnectRequest,
) -> Result<()>
```

Encode a [`SocketConnectRequest`](#socketconnectrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_connect_request

```rust
pub fn read_socket_connect_request(r: Reader<'_>) -> Result<SocketConnectRequest>
```

Decode a [`SocketConnectRequest`](#socketconnectrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_info

```rust
pub fn write_socket_info(b: Builder<'_>, v: &SocketInfo) -> Result<()>
```

Encode a [`SocketInfo`](#socketinfo) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_info

```rust
pub fn read_socket_info(r: Reader<'_>) -> Result<SocketInfo>
```

Decode a [`SocketInfo`](#socketinfo) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_connect_reply

```rust
pub fn write_socket_connect_reply(
    b: Builder<'_>,
    v: &Result<SocketInfo, PluginError>,
) -> Result<()>
```

Encode a `SocketConnectReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_socket_connect_reply

```rust
pub fn read_socket_connect_reply(r: Reader<'_>) -> Result<Result<SocketInfo, PluginError>>
```

Decode a `SocketConnectReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_socket_list

```rust
pub fn write_socket_list(b: Builder<'_>, v: &SocketList) -> Result<()>
```

Encode a [`SocketList`](#socketlist) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_list

```rust
pub fn read_socket_list(r: Reader<'_>) -> Result<SocketList>
```

Decode a [`SocketList`](#socketlist) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_list_reply

```rust
pub fn write_socket_list_reply(
    b: Builder<'_>,
    v: &Result<SocketList, PluginError>,
) -> Result<()>
```

Encode a `SocketListReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_socket_list_reply

```rust
pub fn read_socket_list_reply(r: Reader<'_>) -> Result<Result<SocketList, PluginError>>
```

Decode a `SocketListReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_socket_message_reply

```rust
pub fn write_socket_message_reply(
    b: Builder<'_>,
    v: &Result<SocketMessageOk, PluginError>,
) -> Result<()>
```

Encode a `SocketMessageReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_socket_message_reply

```rust
pub fn read_socket_message_reply(
    r: Reader<'_>,
) -> Result<Result<SocketMessageOk, PluginError>>
```

Decode a `SocketMessageReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_socket_message_ok

```rust
pub fn write_socket_message_ok(b: Builder<'_>, v: &SocketMessageOk) -> Result<()>
```

Encode a [`SocketMessageOk`](#socketmessageok) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_message_ok

```rust
pub fn read_socket_message_ok(r: Reader<'_>) -> Result<SocketMessageOk>
```

Decode a [`SocketMessageOk`](#socketmessageok) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_message

```rust
pub fn write_socket_message(b: Builder<'_>, v: &SocketMessage) -> Result<()>
```

Encode a [`SocketMessage`](#socketmessage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_message

```rust
pub fn read_socket_message(r: Reader<'_>) -> Result<SocketMessage>
```

Decode a [`SocketMessage`](#socketmessage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_socket_closed

```rust
pub fn write_socket_closed(b: Builder<'_>, v: &SocketClosed) -> Result<()>
```

Encode a [`SocketClosed`](#socketclosed) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_socket_closed

```rust
pub fn read_socket_closed(r: Reader<'_>) -> Result<SocketClosed>
```

Decode a [`SocketClosed`](#socketclosed) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_origin_header

```rust
pub fn write_origin_header(b: Builder<'_>, v: &OriginHeader) -> Result<()>
```

Encode a [`OriginHeader`](#originheader) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_origin_header

```rust
pub fn read_origin_header(r: Reader<'_>) -> Result<OriginHeader>
```

Decode a [`OriginHeader`](#originheader) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_origin_request

```rust
pub fn write_origin_request(b: Builder<'_>, v: &OriginRequest) -> Result<()>
```

Encode a [`OriginRequest`](#originrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_origin_request

```rust
pub fn read_origin_request(r: Reader<'_>) -> Result<OriginRequest>
```

Decode a [`OriginRequest`](#originrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_origin_refusal

```rust
pub fn write_origin_refusal(b: Builder<'_>, v: &OriginRefusal) -> Result<()>
```

Encode a [`OriginRefusal`](#originrefusal) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_origin_refusal

```rust
pub fn read_origin_refusal(r: Reader<'_>) -> Result<OriginRefusal>
```

Decode a [`OriginRefusal`](#originrefusal) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_origin_connected

```rust
pub fn write_origin_connected(b: Builder<'_>, v: &OriginConnected) -> Result<()>
```

Encode a `OriginConnected` union.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_origin_connected

```rust
pub fn read_origin_connected(r: Reader<'_>) -> Result<OriginConnected>
```

Decode a `OriginConnected` union.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_origin_connect_reply

```rust
pub fn write_origin_connect_reply(
    b: Builder<'_>,
    v: &Result<OriginConnected, PluginError>,
) -> Result<()>
```

Encode a `OriginConnectReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_origin_connect_reply

```rust
pub fn read_origin_connect_reply(
    r: Reader<'_>,
) -> Result<Result<OriginConnected, PluginError>>
```

Decode a `OriginConnectReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_ingress_started

```rust
pub fn write_ingress_started(b: Builder<'_>, v: &IngressStarted) -> Result<()>
```

Encode a [`IngressStarted`](#ingressstarted) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ingress_started

```rust
pub fn read_ingress_started(r: Reader<'_>) -> Result<IngressStarted>
```

Decode a [`IngressStarted`](#ingressstarted) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ingress_start_reply

```rust
pub fn write_ingress_start_reply(
    b: Builder<'_>,
    v: &Result<IngressStarted, PluginError>,
) -> Result<()>
```

Encode a `IngressStartReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_ingress_start_reply

```rust
pub fn read_ingress_start_reply(
    r: Reader<'_>,
) -> Result<Result<IngressStarted, PluginError>>
```

Decode a `IngressStartReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_webhook_url

```rust
pub fn write_webhook_url(b: Builder<'_>, v: &WebhookUrl) -> Result<()>
```

Encode a [`WebhookUrl`](#webhookurl) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_url

```rust
pub fn read_webhook_url(r: Reader<'_>) -> Result<WebhookUrl>
```

Decode a [`WebhookUrl`](#webhookurl) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_url_reply

```rust
pub fn write_webhook_url_reply(
    b: Builder<'_>,
    v: &Result<WebhookUrl, PluginError>,
) -> Result<()>
```

Encode a `WebhookUrlReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_webhook_url_reply

```rust
pub fn read_webhook_url_reply(r: Reader<'_>) -> Result<Result<WebhookUrl, PluginError>>
```

Decode a `WebhookUrlReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_webhook_pull_request

```rust
pub fn write_webhook_pull_request(b: Builder<'_>, v: &WebhookPullRequest) -> Result<()>
```

Encode a [`WebhookPullRequest`](#webhookpullrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_pull_request

```rust
pub fn read_webhook_pull_request(r: Reader<'_>) -> Result<WebhookPullRequest>
```

Decode a [`WebhookPullRequest`](#webhookpullrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_header

```rust
pub fn write_webhook_header(b: Builder<'_>, v: &WebhookHeader) -> Result<()>
```

Encode a [`WebhookHeader`](#webhookheader) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_header

```rust
pub fn read_webhook_header(r: Reader<'_>) -> Result<WebhookHeader>
```

Decode a [`WebhookHeader`](#webhookheader) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_delivery

```rust
pub fn write_webhook_delivery(b: Builder<'_>, v: &WebhookDelivery) -> Result<()>
```

Encode a [`WebhookDelivery`](#webhookdelivery) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_delivery

```rust
pub fn read_webhook_delivery(r: Reader<'_>) -> Result<WebhookDelivery>
```

Decode a [`WebhookDelivery`](#webhookdelivery) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_pull_page

```rust
pub fn write_webhook_pull_page(b: Builder<'_>, v: &WebhookPullPage) -> Result<()>
```

Encode a [`WebhookPullPage`](#webhookpullpage) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_pull_page

```rust
pub fn read_webhook_pull_page(r: Reader<'_>) -> Result<WebhookPullPage>
```

Decode a [`WebhookPullPage`](#webhookpullpage) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_pull_reply

```rust
pub fn write_webhook_pull_reply(
    b: Builder<'_>,
    v: &Result<WebhookPullPage, PluginError>,
) -> Result<()>
```

Encode a `WebhookPullReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_webhook_pull_reply

```rust
pub fn read_webhook_pull_reply(
    r: Reader<'_>,
) -> Result<Result<WebhookPullPage, PluginError>>
```

Decode a `WebhookPullReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_ingress_health

```rust
pub fn write_ingress_health(b: Builder<'_>, v: &IngressHealth) -> Result<()>
```

Encode a [`IngressHealth`](#ingresshealth) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_ingress_health

```rust
pub fn read_ingress_health(r: Reader<'_>) -> Result<IngressHealth>
```

Decode a [`IngressHealth`](#ingresshealth) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_ingress_health_reply

```rust
pub fn write_ingress_health_reply(
    b: Builder<'_>,
    v: &Result<IngressHealth, PluginError>,
) -> Result<()>
```

Encode a `IngressHealthReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_ingress_health_reply

```rust
pub fn read_ingress_health_reply(
    r: Reader<'_>,
) -> Result<Result<IngressHealth, PluginError>>
```

Decode a `IngressHealthReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_webhook_ack_request

```rust
pub fn write_webhook_ack_request(b: Builder<'_>, v: &WebhookAckRequest) -> Result<()>
```

Encode a [`WebhookAckRequest`](#webhookackrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_ack_request

```rust
pub fn read_webhook_ack_request(r: Reader<'_>) -> Result<WebhookAckRequest>
```

Decode a [`WebhookAckRequest`](#webhookackrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_webhook_delivered

```rust
pub fn write_webhook_delivered(b: Builder<'_>, v: &WebhookDelivered) -> Result<()>
```

Encode a [`WebhookDelivered`](#webhookdelivered) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_webhook_delivered

```rust
pub fn read_webhook_delivered(r: Reader<'_>) -> Result<WebhookDelivered>
```

Decode a [`WebhookDelivered`](#webhookdelivered) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_provision_grant

```rust
pub fn write_provision_grant(b: Builder<'_>, v: &ProvisionGrant) -> Result<()>
```

Encode a [`ProvisionGrant`](#provisiongrant) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_provision_grant

```rust
pub fn read_provision_grant(r: Reader<'_>) -> Result<ProvisionGrant>
```

Decode a [`ProvisionGrant`](#provisiongrant) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_provision_request

```rust
pub fn write_provision_request(b: Builder<'_>, v: &ProvisionRequest) -> Result<()>
```

Encode a [`ProvisionRequest`](#provisionrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_provision_request

```rust
pub fn read_provision_request(r: Reader<'_>) -> Result<ProvisionRequest>
```

Decode a [`ProvisionRequest`](#provisionrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_provisioned_secret

```rust
pub fn write_provisioned_secret(b: Builder<'_>, v: &ProvisionedSecret) -> Result<()>
```

Encode a [`ProvisionedSecret`](#provisionedsecret) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_provisioned_secret

```rust
pub fn read_provisioned_secret(r: Reader<'_>) -> Result<ProvisionedSecret>
```

Decode a [`ProvisionedSecret`](#provisionedsecret) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_provisioned

```rust
pub fn write_provisioned(b: Builder<'_>, v: &Provisioned) -> Result<()>
```

Encode a [`Provisioned`](#provisioned) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_provisioned

```rust
pub fn read_provisioned(r: Reader<'_>) -> Result<Provisioned>
```

Decode a [`Provisioned`](#provisioned) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_provision_reply

```rust
pub fn write_provision_reply(
    b: Builder<'_>,
    v: &Result<Provisioned, PluginError>,
) -> Result<()>
```

Encode a `ProvisionReply` union from a typed result.

#### Errors

Returns a Cap'n Proto error when a nested field cannot be encoded.

### read_provision_reply

```rust
pub fn read_provision_reply(r: Reader<'_>) -> Result<Result<Provisioned, PluginError>>
```

Decode a `ProvisionReply` union into a typed result.

#### Errors

Returns a Cap'n Proto error when the union is unset or a field is malformed.

### write_deprovision_request

```rust
pub fn write_deprovision_request(b: Builder<'_>, v: &DeprovisionRequest) -> Result<()>
```

Encode a [`DeprovisionRequest`](#deprovisionrequest) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_deprovision_request

```rust
pub fn read_deprovision_request(r: Reader<'_>) -> Result<DeprovisionRequest>
```

Decode a [`DeprovisionRequest`](#deprovisionrequest) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_event_poll_params

```rust
pub fn write_event_poll_params(b: Builder<'_>, v: &EventPollParams) -> Result<()>
```

Encode a [`EventPollParams`](#eventpollparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_event_poll_params

```rust
pub fn read_event_poll_params(r: Reader<'_>) -> Result<EventPollParams>
```

Decode a [`EventPollParams`](#eventpollparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

### write_event_ack_params

```rust
pub fn write_event_ack_params(b: Builder<'_>, v: &EventAckParams) -> Result<()>
```

Encode a [`EventAckParams`](#eventackparams) onto a Cap'n Proto builder.

#### Errors

Returns a Cap'n Proto error when a list is too long or a nested field cannot be encoded.

### read_event_ack_params

```rust
pub fn read_event_ack_params(r: Reader<'_>) -> Result<EventAckParams>
```

Decode a [`EventAckParams`](#eventackparams) from a Cap'n Proto reader.

#### Errors

Returns a Cap'n Proto error when a text field is not UTF-8 or a pointer is malformed.

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
pub const READ_ANY_COUNT: u8 = 181;
```

Number of root structs [`read_any_message`](#read_any_message) can decode; selectors are
`0..READ_ANY_COUNT` in schema order.
