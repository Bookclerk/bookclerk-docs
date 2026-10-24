# bookclerk_plugin_sdk.abi

GENERATED FILE - do not edit. Run `python3 scripts/gen-plugin-abi.py --write` after changing crates/bookclerk-plugin-abi/schema/plugin.capnp.

Author-facing Python projection of every struct, union, enum, and
interface in `crates/bookclerk-plugin-abi/schema/plugin.capnp`.
TypedDict keys are the schema names (camelCase). Cap'n Proto unions
become `{"kind": ..., "value": ...}` TypedDict unions; `Void`
members carry no `value`. 64-bit integers are Python `int`; `Data`
is `bytes`. Interfaces are `Protocol` classes with `async` methods.

| Name | Kind | Summary |
| --- | --- | --- |
| [`ScalarLimits`](#scalarlimits) | class | Guest-advertised caps for `rpc.scalarLimits`; never above the file constants. |
| [`PluginError`](#pluginerror) | class | `code` is a snake_case string (`not_found`, `invalid_cursor`, …). Unknown codes are forwarded as-is; SDKs surface them as PluginErrorCode.unknown while retaining the raw wire code. |
| [`ObjectMetadata`](#objectmetadata) | class | Full metadata of one stored object. |
| [`ObjectInfo`](#objectinfo) | class | Compact object entry in a list page. |
| [`ListOptions`](#listoptions) | class | Paging options for `Destination.list`. |
| [`ListPage`](#listpage) | class | One page of `Destination.list` results. |
| [`ByteRange`](#byterange) | class | Half-open byte window `[offset, offset + length)`. |
| [`ReadOptions`](#readoptions) | class | Options for `Destination.get`. |
| [`WriteOptions`](#writeoptions) | class | Options for `Destination.put`. |
| [`PutResult`](#putresult) | class | Summary of a stored (or committed) object. |
| [`CopyResult`](#copyresult) | class | Summary of a server-side copy. |
| [`PluginDescribe`](#plugindescribe) | class | Guest identity and negotiation surface returned by `describe()`. |
| [`OidcClientTemplate`](#oidcclienttemplate) | class | Bookclerk-as-IdP relying-party template. Plugins declare callback path and client id; the host materializes `oidc_clients` rows and remains the AS. `originConfigKey` is a dotted config path (e.g. integrations.audiobookshelf.base_url). |
| [`OidcClientsOk`](#oidcclientsok) | class | Success payload of `Oidc.clients`. |
| [`OidcClientsReplyOk`](#oidcclientsreplyok) | class | `OidcClientsReply` member `ok`. |
| [`OidcClientsReplyErr`](#oidcclientsreplyerr) | class | `OidcClientsReply` member `err`. |
| [`ExtensibleConfig`](#extensibleconfig) | class | Plugin-specific extensible config. Not a substitute for typed ABI fields. |
| [`Bindings`](#bindings) | class | Host-granted bindings for one `PluginWorker.open` (`env` in the Workers idiom). OS paths, FDs, and sockets are transport-private. Capability fields are null when the manifest does not declare (or the operator did not grant) the binding. |
| [`Entrypoints`](#entrypoints) | class | Exported entrypoints returned by `PluginWorker.open`. One capability per `plugin.toml` `entrypoints` entry / trigger; null when not exported. The host refuses an entrypoint the manifest or operator grant did not allow. |
| [`EntrypointsReplyOk`](#entrypointsreplyok) | class | `EntrypointsReply` member `ok`. |
| [`EntrypointsReplyErr`](#entrypointsreplyerr) | class | `EntrypointsReply` member `err`. |
| [`JobInvocation`](#jobinvocation) | class | Durable command envelope (not a domain event). Command payload schema version and checkpoint schema versions are independent of the plugin ABI. The envelope itself is not persisted as opaque bytes across ABI majors. Idempotency keys are scoped to (account, plugin, commandType) until a terminal fenced outcome is committed. |
| [`CompletedOutcome`](#completedoutcome) | class | Job finished successfully. |
| [`RetryableOutcome`](#retryableoutcome) | class | Job failed transiently; the host reschedules it. |
| [`RejectedOutcome`](#rejectedoutcome) | class | Job failed permanently; no retry. |
| [`CancelledOutcome`](#cancelledoutcome) | class | Job observed cancellation and stopped. |
| [`SuspendedOutcome`](#suspendedoutcome) | class | Job released the process and asks to be resumed later. |
| [`JobOutcomeCompleted`](#joboutcomecompleted) | class | `JobOutcome` member `completed`. |
| [`JobOutcomeRetryable`](#joboutcomeretryable) | class | `JobOutcome` member `retryable`. |
| [`JobOutcomeRejected`](#joboutcomerejected) | class | `JobOutcome` member `rejected`. |
| [`JobOutcomeCancelled`](#joboutcomecancelled) | class | `JobOutcome` member `cancelled`. |
| [`JobOutcomeSuspended`](#joboutcomesuspended) | class | `JobOutcome` member `suspended`. |
| [`DomainEvent`](#domainevent) | class | Domain event (not a job). Outbox-produced, at-least-once, idempotent consume. |
| [`EventAck`](#eventack) | class | Event handled; the host marks it delivered. |
| [`EventRetry`](#eventretry) | class | Redeliver later. |
| [`EventReject`](#eventreject) | class | Event rejected; the host records the reason and stops delivering. |
| [`EventDeadLetter`](#eventdeadletter) | class | Event moved to the dead-letter queue for operator review. |
| [`EventSuspended`](#eventsuspended) | class | Append-only. Mirrors job SuspendedOutcome; event handlers persist a bounded checkpoint and release the process until wakeAtUnixMs. Optional wake-on-matching-event fields (empty = timestamp-only). |
| [`EventResultAck`](#eventresultack) | class | `EventResult` member `ack`. |
| [`EventResultRetry`](#eventresultretry) | class | `EventResult` member `retry`. |
| [`EventResultReject`](#eventresultreject) | class | `EventResult` member `reject`. |
| [`EventResultDeadLetter`](#eventresultdeadletter) | class | `EventResult` member `deadLetter`. |
| [`EventResultSuspended`](#eventresultsuspended) | class | `EventResult` member `suspended`. |
| [`EventBatch`](#eventbatch) | class | One `EventConsumer.event` delivery: at most `maxListPage` events, each bounded by `maxEventPayloadBytes` / `maxCheckpointBytes`. |
| [`EventBatchReplyOk`](#eventbatchreplyok) | class | `EventBatchReply` member `ok`. |
| [`EventBatchReplyErr`](#eventbatchreplyerr) | class | `EventBatchReply` member `err`. |
| [`JobController`](#jobcontroller) | class | Everything one `JobRunner.job` invocation may touch. Capabilities are host-served; the guest never sees OS paths or sockets. |
| [`HeadOk`](#headok) | class | Success payload of `Destination.head`. |
| [`HeadReplyOk`](#headreplyok) | class | `HeadReply` member `ok`. |
| [`HeadReplyErr`](#headreplyerr) | class | `HeadReply` member `err`. |
| [`ListReplyOk`](#listreplyok) | class | `ListReply` member `ok`. |
| [`ListReplyErr`](#listreplyerr) | class | `ListReply` member `err`. |
| [`GetOk`](#getok) | class | Success payload of `Destination.get`. |
| [`GetReplyOk`](#getreplyok) | class | `GetReply` member `ok`. |
| [`GetReplyErr`](#getreplyerr) | class | `GetReply` member `err`. |
| [`PutReplyOk`](#putreplyok) | class | `PutReply` member `ok`. |
| [`PutReplyErr`](#putreplyerr) | class | `PutReply` member `err`. |
| [`CopyReplyOk`](#copyreplyok) | class | `CopyReply` member `ok`. |
| [`CopyReplyErr`](#copyreplyerr) | class | `CopyReply` member `err`. |
| [`EmptyReplyOk`](#emptyreplyok) | class | `EmptyReply` member `ok`. |
| [`EmptyReplyErr`](#emptyreplyerr) | class | `EmptyReply` member `err`. |
| [`PullOk`](#pullok) | class | Success payload of `ByteSource.pull`. |
| [`PullReplyOk`](#pullreplyok) | class | `PullReply` member `ok`. |
| [`PullReplyErr`](#pullreplyerr) | class | `PullReply` member `err`. |
| [`OpenOk`](#openok) | class | Success payload of `Source.open`. |
| [`OpenReplyOk`](#openreplyok) | class | `OpenReply` member `ok`. |
| [`OpenReplyErr`](#openreplyerr) | class | `OpenReply` member `err`. |
| [`DescribeReplyOk`](#describereplyok) | class | `DescribeReply` member `ok`. |
| [`DescribeReplyErr`](#describereplyerr) | class | `DescribeReply` member `err`. |
| [`HandleReplyOk`](#handlereplyok) | class | `HandleReply` member `ok`. |
| [`HandleReplyErr`](#handlereplyerr) | class | `HandleReply` member `err`. |
| [`HealthOk`](#healthok) | class | Typed liveness report. |
| [`HealthReplyOk`](#healthreplyok) | class | `HealthReply` member `ok`. |
| [`HealthReplyErr`](#healthreplyerr) | class | `HealthReply` member `err`. |
| [`AdapterSessionReplyOk`](#adaptersessionreplyok) | class | `AdapterSessionReply` member `ok`. |
| [`AdapterSessionReplyErr`](#adaptersessionreplyerr) | class | `AdapterSessionReply` member `err`. |
| [`GuestDatabaseReplyOk`](#guestdatabasereplyok) | class | `GuestDatabaseReply` member `ok`. |
| [`GuestDatabaseReplyErr`](#guestdatabasereplyerr) | class | `GuestDatabaseReply` member `err`. |
| [`ByteSource`](#bytesource) | class | Transferred readable byte stream. The capability *is* the stream; callers pull bounded windows. Abort is capability drop / RPC cancel. A failed pull MUST set `err` — never a successful empty EOF. |
| [`Destination`](#destination) | class | Object store the host writes acquired media into (`output.*` plugins). Keys are relative object paths; the plugin owns the physical layout. |
| [`Source`](#source) | class | Read-only byte source for job inputs (not a storefront). |
| [`ProgressSink`](#progresssink) | class | Host-side progress reporter handed to job handlers. |
| [`Cancellation`](#cancellation) | class | Transport cancellation. SDKs project this into a locally created AbortSignal (AbortSignal is not a serializable Workers RPC value). |
| [`JobRunner`](#jobrunner) | class | `[triggers] jobs` handler (`Entrypoints.jobRunner`); runs one durable command. |
| [`EventConsumer`](#eventconsumer) | class | `[[events.consumers]]` handler (`Entrypoints.eventConsumer`). Delivery is at-least-once; consumers must be idempotent on `deduplicationKey`. |
| [`EventPublisher`](#eventpublisher) | class | `EVENTS` binding: host-served outbox publisher granted through `Bindings.events`. Event types must be listed in `[[events.producers]]`. |
| [`NamedDatabase`](#nameddatabase) | class | One named plugin-owned database binding delivered on `Bindings.databases`. |
| [`QueueConsumer`](#queueconsumer) | class | `[[queues.consumers]]` handler (`Entrypoints.queueConsumer`). Delivery is at-least-once; consumers must be idempotent on `QueueMessage.id`. |
| [`QueueProducer`](#queueproducer) | class | Queue producer binding: host-served send into the queue a `[[queues.producers]]` row names, granted through `Bindings.queues`. |
| [`NamedQueue`](#namedqueue) | class | One named queue producer delivered on `Bindings.queues`. |
| [`KvNamespace`](#kvnamespace) | class | Durable key/value namespace: a Workers `KVNamespace`-shaped binding the host serves from the library database, scoped to the plugin and the namespace a `[[kv_namespaces]]` row names, granted through `Bindings.kv`. Values live in the cluster, so an instance moved to another host reads its own keys. |
| [`NamedKv`](#namedkv) | class | One named durable namespace delivered on `Bindings.kv`. |
| [`ContentSource`](#contentsource) | class | Storefront content source (not byte Source). Every method takes and returns typed structs from the "Typed method payloads" section. |
| [`RemoteLibrary`](#remotelibrary) | class | `remoteLibrary` entrypoint: long-running remote-library lifecycle (start / stop, library rescan, listening sync, external-user polling). Event delivery is `EventConsumer`; credential verification is `Oidc`. |
| [`PluginCli`](#plugincli) | class | `cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`. |
| [`Oidc`](#oidc) | class | `oidc` entrypoint: relying-party client templates and credential verification on behalf of the host authorization server. |
| [`EventConsumerSpec`](#eventconsumerspec) | class | One declared event consumer: a `[[events.consumers]]` row the default entrypoint's `event(batch)` handler accepts. |
| [`PluginCapabilities`](#plugincapabilities) | class | Typed capability declaration returned by `describe()`. The host compares it with `plugin.toml` and the operator grant; widening is rejected at spawn. |
| [`Brand`](#brand) | class | Portal brand crossing the RPC boundary. Distinct from `plugin.toml` `logo`: `iconUrl` is the live URL or data URI the SPA renders. |
| [`ConfigOption`](#configoption) | class | One discoverable config option group advertised for sources. |
| [`ConfigOptionValue`](#configoptionvalue) | class | One selectable value under a `ConfigOption`. |
| [`CliSchema`](#clischema) | class | Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`). |
| [`CliCommandSpec`](#clicommandspec) | class | One plugin CLI command under `CliSchema`. |
| [`CliArgSpec`](#cliargspec) | class | One CLI argument or flag under a `CliCommandSpec`. |
| [`CliArg`](#cliarg) | class | One named argument value passed to `PluginCli.invoke`. |
| [`CliInvokeParams`](#cliinvokeparams) | class | Params of `PluginCli.invoke`. |
| [`CliInvokeResult`](#cliinvokeresult) | class | Result of `PluginCli.invoke`. |
| [`CliSchemaReplyOk`](#clischemareplyok) | class | `CliSchemaReply` member `ok`. |
| [`CliSchemaReplyErr`](#clischemareplyerr) | class | `CliSchemaReply` member `err`. |
| [`CliInvokeReplyOk`](#cliinvokereplyok) | class | `CliInvokeReply` member `ok`. |
| [`CliInvokeReplyErr`](#cliinvokereplyerr) | class | `CliInvokeReply` member `err`. |
| [`DatabaseAdapterConfig`](#databaseadapterconfig) | class | Author-facing database adapter configuration carried in `DatabaseContext.adapter`. This is the generic bootstrap mechanism for third-party adapters: the operator's granted `[database.<id>]` table plus the scoped writable data dir. First-party host-managed adapters receive host-private connect params in `DatabaseContext.config` instead. |
| [`DiagnoseResult`](#diagnoseresult) | class | Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`. |
| [`DiagnoseReplyOk`](#diagnosereplyok) | class | `DiagnoseReply` member `ok`. |
| [`DiagnoseReplyErr`](#diagnosereplyerr) | class | `DiagnoseReply` member `err`. |
| [`SourceAccount`](#sourceaccount) | class | Source account metadata returned from login and stored by the host. |
| [`LoginParams`](#loginparams) | class | Params of `ContentSource.login` and `ContentSource.loginStart`. Password sources fill email/password; OAuth sources use callback / external fields. There is no files-dir root or library DB path -- only `pluginDataDir`. |
| [`LoginResult`](#loginresult) | class | Result of `ContentSource.login` / `loginComplete`: account metadata plus opaque credentials for the host to seal into `encrypted_secrets` (`provider = plugin id`). Guests never write secrets into the library DB. |
| [`LoginStartResult`](#loginstartresult) | class | Result of `ContentSource.loginStart` (interactive OAuth). The operator opens `url`; `loginComplete` later uses `sessionId`. |
| [`LoginCompleteParams`](#logincompleteparams) | class | Params of `ContentSource.loginComplete`. |
| [`LoginReplyOk`](#loginreplyok) | class | `LoginReply` member `ok`. |
| [`LoginReplyErr`](#loginreplyerr) | class | `LoginReply` member `err`. |
| [`LoginStartReplyOk`](#loginstartreplyok) | class | `LoginStartReply` member `ok`. |
| [`LoginStartReplyErr`](#loginstartreplyerr) | class | `LoginStartReply` member `err`. |
| [`AccountCredential`](#accountcredential) | class | Host-sealed credentials for one account, delivered on `scan`. |
| [`ScanParams`](#scanparams) | class | Params of `ContentSource.scan`. The host injects sealed credentials so the plugin does not need a private credential store under `pluginDataDir`. |
| [`ScanBook`](#scanbook) | class | One library title returned by `ContentSource.scan`. The host upserts these rows and forces `source` to the plugin id. |
| [`ScanSummary`](#scansummary) | class | Summary result of `ContentSource.scan`. |
| [`ScanReplyOk`](#scanreplyok) | class | `ScanReply` member `ok`. |
| [`ScanReplyErr`](#scanreplyerr) | class | `ScanReply` member `err`. |
| [`FetchOptions`](#fetchoptions) | class | Fetch-relevant acquire knobs the host forwards so external load matches in-process. Packaging / naming knobs stay host-side. |
| [`FetchTitleParams`](#fetchtitleparams) | class | Params of `ContentSource.fetchTitle`. The plugin writes media under `cacheDir` and returns plain (DRM-free) paths. The host injects credentials; guests must not open `library.db` or `master.key`. |
| [`PlainPart`](#plainpart) | class | One plain audio part written under the cache directory. |
| [`ChapterMarker`](#chaptermarker) | class | One chapter marker of a fetched title. |
| [`PlainFetch`](#plainfetch) | class | Plain (DRM-free) fetch result. Sources always return decrypted media; DRM guests decrypt before responding. |
| [`FetchTitleReplyOk`](#fetchtitlereplyok) | class | `FetchTitleReply` member `ok`. |
| [`FetchTitleReplyErr`](#fetchtitlereplyerr) | class | `FetchTitleReply` member `err`. |
| [`SourceAccounts`](#sourceaccounts) | class | Success payload of `ContentSource.listAccounts`. |
| [`SourceAccountsReplyOk`](#sourceaccountsreplyok) | class | `SourceAccountsReply` member `ok`. |
| [`SourceAccountsReplyErr`](#sourceaccountsreplyerr) | class | `SourceAccountsReply` member `err`. |
| [`SearchCatalogParams`](#searchcatalogparams) | class | Params of `ContentSource.searchCatalog`. |
| [`ExpandCandidatesParams`](#expandcandidatesparams) | class | Params of `ContentSource.expandCandidates`. Seed fields identify a known title; the guest returns related catalog hits. |
| [`PurchaseHintParams`](#purchasehintparams) | class | Params of `ContentSource.purchaseHint`. At least one identity field (`productId` / `asin` / `isbn` / title+authors) should be set; guests may return `invalid_params` when none are usable. |
| [`ListDealsParams`](#listdealsparams) | class | Params of `ContentSource.listDeals`. |
| [`CatalogDetailParams`](#catalogdetailparams) | class | Params of `ContentSource.catalogDetail`. |
| [`RevokeAccountParams`](#revokeaccountparams) | class | Params of `ContentSource.revokeAccount`. The host sends the credentials it is about to delete so the guest can sign the device out at the store. |
| [`CatalogHit`](#cataloghit) | class | One catalog / candidate hit returned by `searchCatalog`, `expandCandidates`, `listDeals`, and `catalogDetail`. |
| [`CatalogHits`](#cataloghits) | class | Success payload of the catalog list methods. |
| [`CatalogHitsReplyOk`](#cataloghitsreplyok) | class | `CatalogHitsReply` member `ok`. |
| [`CatalogHitsReplyErr`](#cataloghitsreplyerr) | class | `CatalogHitsReply` member `err`. |
| [`CatalogDetail`](#catalogdetail) | class | Success payload of `ContentSource.catalogDetail`. |
| [`CatalogDetailReplyOk`](#catalogdetailreplyok) | class | `CatalogDetailReply` member `ok`. |
| [`CatalogDetailReplyErr`](#catalogdetailreplyerr) | class | `CatalogDetailReply` member `err`. |
| [`PurchaseHint`](#purchasehint) | class | Purchase hint for SPA / CLI purchase deep-links. |
| [`PurchaseHintResult`](#purchasehintresult) | class | Success payload of `ContentSource.purchaseHint`. |
| [`PurchaseHintReplyOk`](#purchasehintreplyok) | class | `PurchaseHintReply` member `ok`. |
| [`PurchaseHintReplyErr`](#purchasehintreplyerr) | class | `PurchaseHintReply` member `err`. |
| [`ScanLibraryParams`](#scanlibraryparams) | class | Params of `RemoteLibrary.scanLibrary` (remote library sync). |
| [`AuthenticateUserParams`](#authenticateuserparams) | class | Params of `Oidc.authenticateUser`. |
| [`ExternalUser`](#externaluser) | class | One external user observed by an integration. The host may mint claim tickets without exposing portal details to the guest. |
| [`ExternalUserReplyOk`](#externaluserreplyok) | class | `ExternalUserReply` member `ok`. |
| [`ExternalUserReplyErr`](#externaluserreplyerr) | class | `ExternalUserReply` member `err`. |
| [`EventPollResult`](#eventpollresult) | class | Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick off workflows. |
| [`EventPollReplyOk`](#eventpollreplyok) | class | `EventPollReply` member `ok`. |
| [`EventPollReplyErr`](#eventpollreplyerr) | class | `EventPollReply` member `err`. |
| [`ListeningProgress`](#listeningprogress) | class | One listening-progress row. The host upserts into `listening_progress` tagged with the plugin id; plugins never open the library DB. |
| [`SyncListeningResult`](#synclisteningresult) | class | Success payload of `RemoteLibrary.syncListening`. |
| [`SyncListeningReplyOk`](#synclisteningreplyok) | class | `SyncListeningReply` member `ok`. |
| [`SyncListeningReplyErr`](#synclisteningreplyerr) | class | `SyncListeningReply` member `err`. |
| [`PluginEvent`](#pluginevent) | class | Guest-published domain event (`EventPublisher.publish`). The host stamps `eventId`, `source` (the plugin id), and `accountId` from the invocation; guests cannot forge either. |
| [`PublishOk`](#publishok) | class | Success payload of `EventPublisher.publish`. |
| [`PublishReplyOk`](#publishreplyok) | class | `PublishReply` member `ok`. |
| [`PublishReplyErr`](#publishreplyerr) | class | `PublishReply` member `err`. |
| [`Invocation`](#invocation) | class | Identity of one `PluginWorker.open` invocation. The host issues ids; guests echo `correlationId` / `causationId` onto published events. |
| [`QueueMessage`](#queuemessage) | class | One message on a Workers-style queue, delivered by `QueueConsumer.queue`. |
| [`QueueBatch`](#queuebatch) | class | One `QueueConsumer.queue` delivery: messages from one queue in send order. |
| [`QueueRetry`](#queueretry) | class | Redeliver a queue message later. |
| [`QueueResultAck`](#queueresultack) | class | `QueueResult` member `ack`. |
| [`QueueResultRetry`](#queueresultretry) | class | `QueueResult` member `retry`. |
| [`QueueBatchOk`](#queuebatchok) | class | Success payload of `QueueConsumer.queue`: exactly one entry per `QueueBatch.messages` entry, in order. A short list is a host-side error and the missing tail is retried. |
| [`QueueBatchReplyOk`](#queuebatchreplyok) | class | `QueueBatchReply` member `ok`. |
| [`QueueBatchReplyErr`](#queuebatchreplyerr) | class | `QueueBatchReply` member `err`. |
| [`QueueSendMessage`](#queuesendmessage) | class | One message to send through a `[[queues.producers]]` binding. |
| [`QueueSendOk`](#queuesendok) | class | Success payload of `QueueProducer.send` / `sendBatch`. |
| [`QueueSendReplyOk`](#queuesendreplyok) | class | `QueueSendReply` member `ok`. |
| [`QueueSendReplyErr`](#queuesendreplyerr) | class | `QueueSendReply` member `err`. |
| [`KvPutOptions`](#kvputoptions) | class | Expiration and metadata for `KvNamespace.put`. |
| [`KvGetOk`](#kvgetok) | class | Success payload of `KvNamespace.get`. |
| [`KvGetReplyOk`](#kvgetreplyok) | class | `KvGetReply` member `ok`. |
| [`KvGetReplyErr`](#kvgetreplyerr) | class | `KvGetReply` member `err`. |
| [`KvPutOk`](#kvputok) | class | Success payload of `KvNamespace.put`. |
| [`KvDeleteOk`](#kvdeleteok) | class | Success payload of `KvNamespace.delete`. |
| [`KvPutReplyOk`](#kvputreplyok) | class | `KvPutReply` member `ok`. |
| [`KvPutReplyErr`](#kvputreplyerr) | class | `KvPutReply` member `err`. |
| [`KvDeleteReplyOk`](#kvdeletereplyok) | class | `KvDeleteReply` member `ok`. |
| [`KvDeleteReplyErr`](#kvdeletereplyerr) | class | `KvDeleteReply` member `err`. |
| [`KvListOptions`](#kvlistoptions) | class | Page request for `KvNamespace.list`. |
| [`KvListKey`](#kvlistkey) | class | One key on a `KvNamespace.list` page. |
| [`KvListOk`](#kvlistok) | class | Success payload of `KvNamespace.list`. |
| [`KvListReplyOk`](#kvlistreplyok) | class | `KvListReply` member `ok`. |
| [`KvListReplyErr`](#kvlistreplyerr) | class | `KvListReply` member `err`. |
| [`DbValueNull`](#dbvaluenull) | class | `DbValue` member `null`. |
| [`DbValueBoolean`](#dbvalueboolean) | class | `DbValue` member `boolean`. |
| [`DbValueInt64`](#dbvalueint64) | class | `DbValue` member `int64`. |
| [`DbValueFloat64`](#dbvaluefloat64) | class | `DbValue` member `float64`. |
| [`DbValueText`](#dbvaluetext) | class | `DbValue` member `text`. |
| [`DbValueBytes`](#dbvaluebytes) | class | `DbValue` member `bytes`. |
| [`DbColumn`](#dbcolumn) | class | Result column descriptor. |
| [`DbRow`](#dbrow) | class | One result row. |
| [`DbStatement`](#dbstatement) | class | One guest statement in an `ExecuteRequest`. |
| [`ExecuteRequest`](#executerequest) | class | Guest statement batch for `GuestDatabase.execute`; runs atomically. |
| [`SqlSpan`](#sqlspan) | class | Byte span in the exact canonical SQL string a proof is bound to. |
| [`TextCollateSite`](#textcollatesite) | class | One TEXT expression the adapter must collate bytewise (`COLLATE "C"`). |
| [`IntegerArithSite`](#integerarithsite) | class | One INTEGER arithmetic expression that must not wrap or error on overflow. |
| [`PhysicalAccess`](#physicalaccess) | class | Physical table/column access used for authorization. |
| [`ResolvedAssignment`](#resolvedassignment) | class | Destination assignment `lhs = rhs` (INSERT, UPDATE, ...). |
| [`NamedSqlType`](#namedsqltype) | class | Column name paired with its resolved type. |
| [`ColumnReference`](#columnreference) | class | `REFERENCES` target of one column. |
| [`OptionalColumnReferenceNone`](#optionalcolumnreferencenone) | class | `OptionalColumnReference` member `none`. |
| [`OptionalColumnReferenceSome`](#optionalcolumnreferencesome) | class | `OptionalColumnReference` member `some`. |
| [`ForeignKeyConstraint`](#foreignkeyconstraint) | class | Table-level `FOREIGN KEY` constraint. |
| [`TableConstraintPrimaryKey`](#tableconstraintprimarykey) | class | `TableConstraint` member `primaryKey`. |
| [`TableConstraintUnique`](#tableconstraintunique) | class | `TableConstraint` member `unique`. |
| [`TableConstraintCheck`](#tableconstraintcheck) | class | `TableConstraint` member `check`. |
| [`TableConstraintForeignKey`](#tableconstraintforeignkey) | class | `TableConstraint` member `foreignKey`. |
| [`CreateTableSchema`](#createtableschema) | class | Parsed `CREATE TABLE` (canonical SQL v1). Per-column lists align with `columns`. |
| [`SchemaCreate`](#schemacreate) | class | `CREATE TABLE` action with its durable fingerprint. |
| [`SchemaActionNone`](#schemaactionnone) | class | `SchemaAction` member `none`. |
| [`SchemaActionCreate`](#schemaactioncreate) | class | `SchemaAction` member `create`. |
| [`SchemaActionDrop`](#schemaactiondrop) | class | `SchemaAction` member `drop`. |
| [`ResolvedStatement`](#resolvedstatement) | class | Host-produced typed proof bound to one exact canonical statement. |
| [`AdapterReceipt`](#adapterreceipt) | class | Replay receipt the host asks the adapter to persist with the batch. |
| [`AdapterStatement`](#adapterstatement) | class | One host-lowered statement with its proof. |
| [`AdapterExecuteRequest`](#adapterexecuterequest) | class | Host -> adapter batch: proven statements plus isolation and receipt. |
| [`StatementResult`](#statementresult) | class | Outcome of one statement. |
| [`DbTiming`](#dbtiming) | class | Engine timing on `ExecuteReply`. |
| [`ExecuteReply`](#executereply) | class | Success payload of `execute`. |
| [`ExecuteResultReplyOk`](#executeresultreplyok) | class | `ExecuteResultReply` member `ok`. |
| [`ExecuteResultReplyErr`](#executeresultreplyerr) | class | `ExecuteResultReply` member `err`. |
| [`DbCapabilities`](#dbcapabilities) | class | Semantic SQL-contract advertisement. Diagnostic engine identity is not part of the capability plane — see `DbBootstrap`. |
| [`DbBootstrapReplyOk`](#dbbootstrapreplyok) | class | `DbBootstrapReply` member `ok`. |
| [`DbBootstrapReplyErr`](#dbbootstrapreplyerr) | class | `DbBootstrapReply` member `err`. |
| [`DbBootstrap`](#dbbootstrap) | class | Bootstrap-only diagnostic metadata (not a capability). |
| [`DbCapabilitiesReplyOk`](#dbcapabilitiesreplyok) | class | `DbCapabilitiesReply` member `ok`. |
| [`DbCapabilitiesReplyErr`](#dbcapabilitiesreplyerr) | class | `DbCapabilitiesReply` member `err`. |
| [`Database`](#database) | class | `databaseAdapter` entrypoint (`Entrypoints.databaseAdapter`). |
| [`IdentityHighWater`](#identityhighwater) | class | Adapter-private identity high-water (sqlite_sequence / bookclerk_identity). Column names live in the canonical backup schema, not this catalog. |
| [`IdentityExportReplyOk`](#identityexportreplyok) | class | `IdentityExportReply` member `ok`. |
| [`IdentityExportReplyErr`](#identityexportreplyerr) | class | `IdentityExportReply` member `err`. |
| [`UserRelationsReplyOk`](#userrelationsreplyok) | class | `UserRelationsReply` member `ok`. |
| [`UserRelationsReplyErr`](#userrelationsreplyerr) | class | `UserRelationsReply` member `err`. |
| [`AdapterDatabaseSession`](#adapterdatabasesession) | class | Host ↔ database adapter plugin. Capability negotiation + typed execute only. |
| [`GuestDatabase`](#guestdatabase) | class | Host-granted SQL for job plugin authors. SDK `DatabaseBinding` mirrors the Cloudflare Workers D1 surface (`prepare`/`bind`/`run`/`first`/`all`/`raw`, `batch`, `exec`) over this typed `execute` transport; wire types stay Cap'n `ExecuteRequest`/`ExecuteReply`. |
| [`PluginMigrationOpSchema`](#pluginmigrationopschema) | class | `PluginMigrationOp` member `schema`. |
| [`PluginMigrationOpData`](#pluginmigrationopdata) | class | `PluginMigrationOp` member `data`. |
| [`PluginMigration`](#pluginmigration) | class | One plugin-owned migration application. `id` is an opaque plugin-chosen stable identity (name, UUID, timestamp-like string, or digits-as-text). Bookclerk assigns no order, version, or predecessor meaning to `id`. Registration order is the forward sequence. |
| [`PluginMigrationsOk`](#pluginmigrationsok) | class | Success payload of `PluginWorker.databaseMigrations`. |
| [`PluginMigrationsReplyOk`](#pluginmigrationsreplyok) | class | `PluginMigrationsReply` member `ok`. |
| [`PluginMigrationsReplyErr`](#pluginmigrationsreplyerr) | class | `PluginMigrationsReply` member `err`. |
| [`PluginWorker`](#pluginworker) | class | Plugin bootstrap capability: the guest's root object (Workers `default` export analogue). `describe()` first; `open()` once per invocation. |
| [`PluginErrorCode`](#pluginerrorcode) | constant | Stable `PluginError.code` strings. Unknown future codes are forwarded as-is; SDKs surface them as a local `unknown` while keeping the raw wire code. |
| [`Entrypoint`](#entrypoint) | constant | Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint analogue). Each value is a capability the host calls over RPC; triggers on the default entrypoint (`event`, `job`) are declared separately. |
| [`PortalAuthMode`](#portalauthmode) | constant | Portal Accounts connect mode for storefronts. |
| [`CliArgKind`](#cliargkind) | constant | Value kind for a `CliArgSpec`. |
| [`CatalogSort`](#catalogsort) | constant | Catalog search ordering. |
| [`CatalogField`](#catalogfield) | constant | Catalog search facet restricting which field the query matches. |
| [`Abridgement`](#abridgement) | constant | Whether an edition is abridged. |
| [`DbType`](#dbtype) | constant | Universal database cell/parameter domain. Engine-native arrays, enums, unsigned integers, and JSON text sentinels are not baseline ABI values. |
| [`DbStatementKind`](#dbstatementkind) | constant | How the host classifies a statement for result handling. |
| [`DbResultSelection`](#dbresultselection) | constant | Which part of a statement's outcome the caller wants back. |
| [`IsolationReq`](#isolationreq) | constant | Host → adapter execute. GuestDatabase stays ExecuteRequest-only. |
| [`ResolvedSqlType`](#resolvedsqltype) | constant | Resolved BookclerkSQL column type after type checking. |
| [`IntegerArithKind`](#integerarithkind) | constant | INTEGER `+` `-` `*` `abs` site lowered to overflow -> NULL. |
| [`OidcClientsReply`](#oidcclientsreply) | constant | Result union of `Oidc.clients`. |
| [`EntrypointsReply`](#entrypointsreply) | constant | Result union of `PluginWorker.open`. |
| [`JobOutcome`](#joboutcome) | constant | Terminal or suspended result of `JobRunner.job`. |
| [`EventResult`](#eventresult) | constant | Per-event outcome inside an `EventConsumer.event` batch reply. |
| [`EventBatchReply`](#eventbatchreply) | constant | Result union of `EventConsumer.event`. `ok` has exactly one entry per `EventBatch.events` entry, in order; a short list is a host-side error and the missing tail is retried. |
| [`HeadReply`](#headreply) | constant | Result union of `Destination.head`. |
| [`ListReply`](#listreply) | constant | Result union of `Destination.list`. |
| [`GetReply`](#getreply) | constant | Result union of `Destination.get`. |
| [`PutReply`](#putreply) | constant | Result union of `Destination.put` / `Destination.commit`. |
| [`CopyReply`](#copyreply) | constant | Result union of `Destination.copy`. |
| [`EmptyReply`](#emptyreply) | constant | Result union of `methods without a success payload`. |
| [`PullReply`](#pullreply) | constant | Result union of `ByteSource.pull`. |
| [`OpenReply`](#openreply) | constant | Result union of `Source.open`. |
| [`DescribeReply`](#describereply) | constant | Result union of `PluginWorker.describe`. |
| [`HandleReply`](#handlereply) | constant | Result union of `JobRunner.job`. |
| [`HealthReply`](#healthreply) | constant | Result union of `health`. |
| [`AdapterSessionReply`](#adaptersessionreply) | constant | Result union of `Database.openSession`. |
| [`GuestDatabaseReply`](#guestdatabasereply) | constant | Result union of `guest database opens`. |
| [`CliSchemaReply`](#clischemareply) | constant | Result union of `PluginCli.describe`. |
| [`CliInvokeReply`](#cliinvokereply) | constant | Result union of `PluginCli.invoke`. |
| [`DiagnoseReply`](#diagnosereply) | constant | Result union of `diagnose`. |
| [`LoginReply`](#loginreply) | constant | Result union of `ContentSource.login` / `loginComplete`. |
| [`LoginStartReply`](#loginstartreply) | constant | Result union of `ContentSource.loginStart`. |
| [`ScanReply`](#scanreply) | constant | Result union of `ContentSource.scan`. |
| [`FetchTitleReply`](#fetchtitlereply) | constant | Result union of `ContentSource.fetchTitle`. |
| [`SourceAccountsReply`](#sourceaccountsreply) | constant | Result union of `ContentSource.listAccounts`. |
| [`CatalogHitsReply`](#cataloghitsreply) | constant | Result union of `searchCatalog` / `expandCandidates` / `listDeals`. |
| [`CatalogDetailReply`](#catalogdetailreply) | constant | Result union of `ContentSource.catalogDetail`. |
| [`PurchaseHintReply`](#purchasehintreply) | constant | Result union of `ContentSource.purchaseHint`. |
| [`ExternalUserReply`](#externaluserreply) | constant | Result union of `Oidc.authenticateUser`. |
| [`EventPollReply`](#eventpollreply) | constant | Result union of `RemoteLibrary.pollEvents`. |
| [`SyncListeningReply`](#synclisteningreply) | constant | Result union of `RemoteLibrary.syncListening`. |
| [`PublishReply`](#publishreply) | constant | Result union of `EventPublisher.publish`. |
| [`QueueResult`](#queueresult) | constant | Per-message outcome inside a `QueueConsumer.queue` batch reply. |
| [`QueueBatchReply`](#queuebatchreply) | constant | Result union of `QueueConsumer.queue`. |
| [`QueueSendReply`](#queuesendreply) | constant | Result union of `QueueProducer.send` / `sendBatch`. |
| [`KvGetReply`](#kvgetreply) | constant | Result union of `KvNamespace.get`. |
| [`KvPutReply`](#kvputreply) | constant | Result union of `KvNamespace.put`. |
| [`KvDeleteReply`](#kvdeletereply) | constant | Result union of `KvNamespace.delete`. |
| [`KvListReply`](#kvlistreply) | constant | Result union of `KvNamespace.list`. |
| [`DbValue`](#dbvalue) | constant | One typed SQL cell or bind parameter. |
| [`OptionalColumnReference`](#optionalcolumnreference) | constant | Per-column `REFERENCES` slot in `CreateTableSchema`. |
| [`TableConstraint`](#tableconstraint) | constant | One table-level constraint. |
| [`SchemaAction`](#schemaaction) | constant | CREATE/DROP action recorded on a proof. |
| [`ExecuteResultReply`](#executeresultreply) | constant | Result union of `execute`. |
| [`DbBootstrapReply`](#dbbootstrapreply) | constant | Result union of `AdapterDatabaseSession.bootstrap`. |
| [`DbCapabilitiesReply`](#dbcapabilitiesreply) | constant | Result union of `AdapterDatabaseSession.capabilities`. |
| [`IdentityExportReply`](#identityexportreply) | constant | Result union of `AdapterDatabaseSession.exportIdentity`. |
| [`UserRelationsReply`](#userrelationsreply) | constant | Result union of `AdapterDatabaseSession.listUserRelations`. |
| [`PluginMigrationOp`](#pluginmigrationop) | constant | One already-separated BookclerkSQL operation in a plugin-owned migration. `schema` is admitted DDL; `data` is admitted DML. There is no native SQL escape hatch. |
| [`PluginMigrationsReply`](#pluginmigrationsreply) | constant | Result union of `PluginWorker.databaseMigrations`. |

## Classes

### ScalarLimits

```python
class ScalarLimits(TypedDict)
```

Guest-advertised caps for `rpc.scalarLimits`; never above the file constants.

**Attributes**

- `maxScalarBytes`: Largest scalar the guest accepts; at most `maxScalarBytes`.
- `maxStreamWindowBytes`: Largest `ByteSource.pull` window; at most `maxStreamWindowBytes`.
- `maxListPage`: Largest list page; at most `maxListPage`.

### PluginError

```python
class PluginError(TypedDict)
```

`code` is a snake_case string (`not_found`, `invalid_cursor`, …). Unknown
codes are forwarded as-is; SDKs surface them as PluginErrorCode.unknown
while retaining the raw wire code.

**Attributes**

- `code`: Stable snake_case error code (see `PluginErrorCode`).
- `message`: Human-readable detail; never contains secrets.

### ObjectMetadata

```python
class ObjectMetadata(TypedDict)
```

Full metadata of one stored object.

**Attributes**

- `key`: Object key.
- `size`: Object size in bytes.
- `contentType`: MIME type; empty when unknown.
- `etag`: Backend entity tag; empty when unsupported.
- `sha256`: Raw SHA-256 digest (32 bytes) or empty when unknown.

### ObjectInfo

```python
class ObjectInfo(TypedDict)
```

Compact object entry in a list page.

**Attributes**

- `key`: Object key.
- `size`: Object size in bytes.

### ListOptions

```python
class ListOptions(TypedDict)
```

Paging options for `Destination.list`.

**Attributes**

- `prefix`: Only keys starting with this prefix; empty lists everything.
- `cursor`: Opaque cursor from a previous `ListPage.nextCursor`; empty starts over.
- `limit`: Requested page size; clamped to `maxListPage`. `0` means guest default.

### ListPage

```python
class ListPage(TypedDict)
```

One page of `Destination.list` results.

**Attributes**

- `objects`: Objects in this page, in backend order.
- `nextCursor`: Cursor for the next page; empty when exhausted.

### ByteRange

```python
class ByteRange(TypedDict)
```

Half-open byte window `[offset, offset + length)`.

**Attributes**

- `offset`: First byte offset.
- `length`: Number of bytes; `0` reads to the end.

### ReadOptions

```python
class ReadOptions(TypedDict)
```

Options for `Destination.get`.

**Attributes**

- `range`: Byte range to read; an all-zero range reads the whole object.

### WriteOptions

```python
class WriteOptions(TypedDict)
```

Options for `Destination.put`.

**Attributes**

- `contentType`: MIME type to record; empty when unknown.
- `contentLength`: Expected body length in bytes; `0` when unknown (chunked).
- `sha256`: Expected raw SHA-256 digest; empty skips verification.
- `commitToken`: Destination-side stage-and-publish. Empty means a one-shot put.
- `stageOnly`: When true, `put` stages remotely and does not publish until `commit`.

### PutResult

```python
class PutResult(TypedDict)
```

Summary of a stored (or committed) object.

**Attributes**

- `key`: Object key written.
- `bytesWritten`: Bytes persisted.
- `etag`: Backend entity tag; empty when unsupported.
- `sha256`: Raw SHA-256 digest of the stored bytes; empty when not computed.

### CopyResult

```python
class CopyResult(TypedDict)
```

Summary of a server-side copy.

**Attributes**

- `bytesCopied`: Bytes copied.

### PluginDescribe

```python
class PluginDescribe(TypedDict)
```

Guest identity and negotiation surface returned by `describe()`.

**Attributes**

- `apiVersion`: ABI version the guest speaks; must equal `apiVersion`.
- `id`: Stable plugin id (`[a-z][a-z0-9_]{0,63}`).
- `displayName`: Human-readable name for UI lists.
- `rpcFeatures`: Negotiable feature names the guest supports (see `feature*` constants).
- `scalarLimits`: Guest caps when `rpc.scalarLimits` is advertised.
- `capabilities`: Exported entrypoints, triggers, and bindings the guest implements. The host rejects anything wider than the manifest and the operator grant.
- `portalAuthMode`: Portal Accounts connect mode for storefronts.
- `passwordEnvVar`: Env var name operators may set for password helpers; never required for Accounts UI connect. Empty when the guest accepts none. Omitted when absent (wire zero value).
- `aliases`: Alternate ids accepted for config / CLI targeting.
- `sortKey`: UI sort weight among peers of the same family; lower sorts first.
- `brand`: Portal brand colors and icon URL; `brand.id` is empty when the guest has no brand and the host renders a neutral fallback.
- `configOptions`: Discoverable config option groups for source UIs.
- `cli`: Embedded CLI schema (same shape as `PluginCli.describe`); empty when unused.

### OidcClientTemplate

```python
class OidcClientTemplate(TypedDict)
```

Bookclerk-as-IdP relying-party template. Plugins declare callback path and
client id; the host materializes `oidc_clients` rows and remains the AS.
`originConfigKey` is a dotted config path (e.g. integrations.audiobookshelf.base_url).

**Attributes**

- `clientId`: OIDC client id the host materializes.
- `displayName`: Name shown on consent / admin screens.
- `callbackPath`: Redirect URI path relative to the integration origin.
- `publicClient`: When true, the client uses PKCE without a client secret.
- `defaultScopes`: Scopes granted by default.
- `issueRefreshToken`: When true, the AS issues refresh tokens to this client.
- `originConfigKey`: Dotted config path holding the client's origin URL.

### OidcClientsOk

```python
class OidcClientsOk(TypedDict)
```

Success payload of `Oidc.clients`.

**Attributes**

- `clients`: Client templates; empty when the plugin is not a relying party.

### OidcClientsReplyOk

```python
class OidcClientsReplyOk(TypedDict)
```

`OidcClientsReply` member `ok`.

Success: OIDC client templates.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: OIDC client templates.

### OidcClientsReplyErr

```python
class OidcClientsReplyErr(TypedDict)
```

`OidcClientsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### ExtensibleConfig

```python
class ExtensibleConfig(TypedDict)
```

Plugin-specific extensible config. Not a substitute for typed ABI fields.

**Attributes**

- `schemaVersion`: Version of `payload`'s schema, owned by the plugin.
- `mediaType`: Media type of `payload` (e.g. `application/json`).
- `payload`: Bounded encoded payload; at most `maxConfigPayloadBytes`.

### Bindings

```python
class Bindings(TypedDict)
```

Host-granted bindings for one `PluginWorker.open` (`env` in the Workers
idiom). OS paths, FDs, and sockets are transport-private. Capability fields
are null when the manifest does not declare (or the operator did not grant)
the binding.

**Attributes**

- `config`: `CONFIG`: granted plugin settings (operator `[vars]` plus the family settings table) as `application/json`; at most `maxConfigPayloadBytes`.
- `secrets`: `SECRETS`: granted secret values as `application/json`; empty payload when the manifest declares no `[secrets]`.
- `adapter`: Database-adapter bootstrap for the `databaseAdapter` entrypoint. First- party host-managed adapters receive host-private connect params in `config`; third-party adapters receive this typed bootstrap (and an empty `config`).
- `events`: `EVENTS`: outbox publisher; null unless `[[events.producers]]` is granted. Null when the guest does not export this capability.
- `databases`: Named plugin-owned `[[databases]]` bindings: each entry is an isolated database provisioned by the active adapter, separate from the Bookclerk library and from every other plugin. Empty when the manifest declares none.
- `cancel`: Host cancellation for the whole invocation (fence / lease loss). Null when the guest does not export this capability.
- `storage`: `WORK_FS`: host-granted object storage for durable plugin files (work filesystem); null unless `[work_fs]` is granted. Job input/output travel on `JobRunner.job(controller)`, never here. Null when the guest does not export this capability.
- `queues`: Named `[[queues.producers]]` bindings: one host-served producer per binding, each sending into the queue the row names. Empty when the manifest declares none.
- `kv`: Named `[[kv_namespaces]]` bindings: one host-served durable namespace per binding, scoped to this plugin. Empty when the manifest declares none.

### Entrypoints

```python
class Entrypoints(TypedDict)
```

Exported entrypoints returned by `PluginWorker.open`. One capability per
`plugin.toml` `entrypoints` entry / trigger; null when not exported. The host
refuses an entrypoint the manifest or operator grant did not allow.

**Attributes**

- `eventConsumer`: `[[events.consumers]]` trigger: `event(batch)` handler. Null when the guest does not export this capability.
- `jobRunner`: `[triggers] jobs` trigger: `job(controller)` handler. Null when the guest does not export this capability.
- `storefront`: `storefront` entrypoint. Null when the guest does not export this capability.
- `storage`: `storage` entrypoint. Null when the guest does not export this capability.
- `databaseAdapter`: `databaseAdapter` entrypoint. Null when the guest does not export this capability.
- `remoteLibrary`: `remoteLibrary` entrypoint. Null when the guest does not export this capability.
- `cli`: `cli` entrypoint. Null when the guest does not export this capability.
- `oidc`: `oidc` entrypoint. Null when the guest does not export this capability.
- `queueConsumer`: `[[queues.consumers]]` trigger: `queue(batch)` handler. Null when the guest does not export this capability.

### EntrypointsReplyOk

```python
class EntrypointsReplyOk(TypedDict)
```

`EntrypointsReply` member `ok`.

Success: exported entrypoint capabilities.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: exported entrypoint capabilities.

### EntrypointsReplyErr

```python
class EntrypointsReplyErr(TypedDict)
```

`EntrypointsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### JobInvocation

```python
class JobInvocation(TypedDict)
```

Durable command envelope (not a domain event). Command payload schema
version and checkpoint schema versions are independent of the plugin ABI.
The envelope itself is not persisted as opaque bytes across ABI majors.
Idempotency keys are scoped to (account, plugin, commandType) until a
terminal fenced outcome is committed.

**Attributes**

- `payloadSchemaVersion`: Schema version of `payloadJson`, owned by the command type.
- `invocationId`: Unique id of this invocation attempt.
- `commandType`: Command type the handler dispatches on.
- `payloadJson`: Command payload (JSON); at most `maxScalarBytes`.
- `idempotencyKey`: Caller idempotency key scoped to (account, plugin, commandType).
- `attempt`: Failure retry counter, starting at 1.
- `correlationId`: Trace correlation id; empty when none.
- `causationId`: Id of the event or command that caused this one; empty when none.
- `deadlineUnixMs`: UTC Unix milliseconds. Host fence/lease is authoritative; this hint must not outlive the fence (clock skew across VPS nodes).
- `checkpointJson`: Checkpoint persisted by a prior `SuspendedOutcome`; empty on first run.
- `checkpointSchemaVersion`: Schema version of `checkpointJson`.
- `invocationSequence`: Resume ordinal; distinct from failure `attempt`.
- `stepId`: Optional step identifier for multi-step commands.

### CompletedOutcome

```python
class CompletedOutcome(TypedDict)
```

Job finished successfully.

**Attributes**

- `message`: Short human summary.
- `bytesCopied`: Bytes produced, when meaningful.

### RetryableOutcome

```python
class RetryableOutcome(TypedDict)
```

Job failed transiently; the host reschedules it.

**Attributes**

- `message`: Short human reason.
- `retryAfterUnixMs`: Earliest retry time; `0` lets the host choose.

### RejectedOutcome

```python
class RejectedOutcome(TypedDict)
```

Job failed permanently; no retry.

**Attributes**

- `message`: Short human reason.

### CancelledOutcome

```python
class CancelledOutcome(TypedDict)
```

Job observed cancellation and stopped.

**Attributes**

- `message`: Short human note.

### SuspendedOutcome

```python
class SuspendedOutcome(TypedDict)
```

Job released the process and asks to be resumed later.

**Attributes**

- `checkpointJson`: Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.
- `checkpointSchemaVersion`: Schema version of `checkpointJson`.
- `wakeAtUnixMs`: Earliest resume time.

### JobOutcomeCompleted

```python
class JobOutcomeCompleted(TypedDict)
```

`JobOutcome` member `completed`.

Finished successfully.

**Attributes**

- `kind`: Always `"completed"`.
- `value`: Finished successfully.

### JobOutcomeRetryable

```python
class JobOutcomeRetryable(TypedDict)
```

`JobOutcome` member `retryable`.

Transient failure; retry later.

**Attributes**

- `kind`: Always `"retryable"`.
- `value`: Transient failure; retry later.

### JobOutcomeRejected

```python
class JobOutcomeRejected(TypedDict)
```

`JobOutcome` member `rejected`.

Permanent failure.

**Attributes**

- `kind`: Always `"rejected"`.
- `value`: Permanent failure.

### JobOutcomeCancelled

```python
class JobOutcomeCancelled(TypedDict)
```

`JobOutcome` member `cancelled`.

Stopped on cancellation.

**Attributes**

- `kind`: Always `"cancelled"`.
- `value`: Stopped on cancellation.

### JobOutcomeSuspended

```python
class JobOutcomeSuspended(TypedDict)
```

`JobOutcome` member `suspended`.

Released with a checkpoint.

**Attributes**

- `kind`: Always `"suspended"`.
- `value`: Released with a checkpoint.

### DomainEvent

```python
class DomainEvent(TypedDict)
```

Domain event (not a job). Outbox-produced, at-least-once, idempotent consume.

**Attributes**

- `eventId`: Unique event id (outbox row identity).
- `eventType`: Dotted event type (e.g. `library.title.added`).
- `schemaVersion`: Schema version of `payload`, owned by the event type.
- `occurredAtUnixMs`: When the producer observed the fact.
- `accountId`: Account scope; empty for host-wide events.
- `correlationId`: Trace correlation id; empty when none.
- `causationId`: Id of the command or event that caused this one; empty when none.
- `deduplicationKey`: Consumer-side idempotency key; stable across redeliveries.
- `deliveryAttempt`: Delivery counter, starting at 1.
- `payload`: Encoded event payload; at most `maxEventPayloadBytes`.
- `checkpointJson`: Append-only. Resume a prior EventResult.suspended.
- `checkpointSchemaVersion`: Schema version of `checkpointJson`.
- `invocationSequence`: Resume ordinal; distinct from `deliveryAttempt`.
- `resumePending`: True when this delivery resumes a prior suspension.
- `source`: Append-only. Producer plugin id; empty when unknown.

### EventAck

```python
class EventAck(TypedDict)
```

Event handled; the host marks it delivered.

**Attributes**

- `dummy`: Placeholder; the struct carries no data.

### EventRetry

```python
class EventRetry(TypedDict)
```

Redeliver later.

**Attributes**

- `retryAtUnixMs`: Earliest redelivery time; `0` lets the host choose.
- `reason`: Short human reason.

### EventReject

```python
class EventReject(TypedDict)
```

Event rejected; the host records the reason and stops delivering.

**Attributes**

- `reason`: Short human reason.

### EventDeadLetter

```python
class EventDeadLetter(TypedDict)
```

Event moved to the dead-letter queue for operator review.

**Attributes**

- `reason`: Short human reason.

### EventSuspended

```python
class EventSuspended(TypedDict)
```

Append-only. Mirrors job SuspendedOutcome; event handlers persist a
bounded checkpoint and release the process until wakeAtUnixMs. Optional
wake-on-matching-event fields (empty = timestamp-only).

**Attributes**

- `checkpointJson`: Bounded checkpoint to replay on resume; at most `maxCheckpointBytes`.
- `checkpointSchemaVersion`: Schema version of `checkpointJson`.
- `wakeAtUnixMs`: Earliest resume time.
- `wakeOnEventType`: Also wake when an event of this type arrives; empty disables.
- `wakeOnFilterJson`: JSON filter applied to matching wake events; empty matches all.

### EventResultAck

```python
class EventResultAck(TypedDict)
```

`EventResult` member `ack`.

Handled.

**Attributes**

- `kind`: Always `"ack"`.
- `value`: Handled.

### EventResultRetry

```python
class EventResultRetry(TypedDict)
```

`EventResult` member `retry`.

Redeliver later.

**Attributes**

- `kind`: Always `"retry"`.
- `value`: Redeliver later.

### EventResultReject

```python
class EventResultReject(TypedDict)
```

`EventResult` member `reject`.

Stop delivering.

**Attributes**

- `kind`: Always `"reject"`.
- `value`: Stop delivering.

### EventResultDeadLetter

```python
class EventResultDeadLetter(TypedDict)
```

`EventResult` member `deadLetter`.

Park for operator review.

**Attributes**

- `kind`: Always `"deadLetter"`.
- `value`: Park for operator review.

### EventResultSuspended

```python
class EventResultSuspended(TypedDict)
```

`EventResult` member `suspended`.

Released with a checkpoint.

**Attributes**

- `kind`: Always `"suspended"`.
- `value`: Released with a checkpoint.

### EventBatch

```python
class EventBatch(TypedDict)
```

One `EventConsumer.event` delivery: at most `maxListPage` events, each
bounded by `maxEventPayloadBytes` / `maxCheckpointBytes`.

**Attributes**

- `events`: Events in delivery order; the reply carries one `EventResult` per entry.

### EventBatchReplyOk

```python
class EventBatchReplyOk(TypedDict)
```

`EventBatchReply` member `ok`.

Success: per-event outcomes.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: per-event outcomes.

### EventBatchReplyErr

```python
class EventBatchReplyErr(TypedDict)
```

`EventBatchReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### JobController

```python
class JobController(TypedDict)
```

Everything one `JobRunner.job` invocation may touch. Capabilities are
host-served; the guest never sees OS paths or sockets.

**Attributes**

- `invocation`: Durable command envelope.
- `input`: Job input objects. Null when the guest does not export this capability.
- `output`: Job output object store. Null when the guest does not export this capability.
- `progress`: Progress reporter (host coalesces frequent updates). Null when the guest does not export this capability.
- `cancel`: Host cancellation probe for this job (fence / lease). Null when the guest does not export this capability.

### HeadOk

```python
class HeadOk(TypedDict)
```

Success payload of `Destination.head`.

**Attributes**

- `found`: Whether the object exists.
- `meta`: Object metadata; meaningful only when `found`.

### HeadReplyOk

```python
class HeadReplyOk(TypedDict)
```

`HeadReply` member `ok`.

Success: head probe outcome.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: head probe outcome.

### HeadReplyErr

```python
class HeadReplyErr(TypedDict)
```

`HeadReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### ListReplyOk

```python
class ListReplyOk(TypedDict)
```

`ListReply` member `ok`.

Success: one list page.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: one list page.

### ListReplyErr

```python
class ListReplyErr(TypedDict)
```

`ListReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### GetOk

```python
class GetOk(TypedDict)
```

Success payload of `Destination.get`.

**Attributes**

- `meta`: Object metadata.
- `body`: Streamed object bytes. Null when the guest does not export this capability.

### GetReplyOk

```python
class GetReplyOk(TypedDict)
```

`GetReply` member `ok`.

Success: object metadata and body stream.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: object metadata and body stream.

### GetReplyErr

```python
class GetReplyErr(TypedDict)
```

`GetReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### PutReplyOk

```python
class PutReplyOk(TypedDict)
```

`PutReply` member `ok`.

Success: stored object summary.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: stored object summary.

### PutReplyErr

```python
class PutReplyErr(TypedDict)
```

`PutReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### CopyReplyOk

```python
class CopyReplyOk(TypedDict)
```

`CopyReply` member `ok`.

Success: copy summary.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: copy summary.

### CopyReplyErr

```python
class CopyReplyErr(TypedDict)
```

`CopyReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### EmptyReplyOk

```python
class EmptyReplyOk(TypedDict)
```

`EmptyReply` member `ok`.

Success: no payload.

**Attributes**

- `kind`: Always `"ok"`.

### EmptyReplyErr

```python
class EmptyReplyErr(TypedDict)
```

`EmptyReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### PullOk

```python
class PullOk(TypedDict)
```

Success payload of `ByteSource.pull`.

**Attributes**

- `chunk`: Next bytes; may be shorter than requested.
- `done`: True when the stream is exhausted after `chunk`.

### PullReplyOk

```python
class PullReplyOk(TypedDict)
```

`PullReply` member `ok`.

Success: one stream window.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: one stream window.

### PullReplyErr

```python
class PullReplyErr(TypedDict)
```

`PullReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### OpenOk

```python
class OpenOk(TypedDict)
```

Success payload of `Source.open`.

**Attributes**

- `meta`: Object metadata.
- `body`: Streamed object bytes. Null when the guest does not export this capability.

### OpenReplyOk

```python
class OpenReplyOk(TypedDict)
```

`OpenReply` member `ok`.

Success: object metadata and body stream.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: object metadata and body stream.

### OpenReplyErr

```python
class OpenReplyErr(TypedDict)
```

`OpenReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### DescribeReplyOk

```python
class DescribeReplyOk(TypedDict)
```

`DescribeReply` member `ok`.

Success: plugin identity.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: plugin identity.

### DescribeReplyErr

```python
class DescribeReplyErr(TypedDict)
```

`DescribeReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### HandleReplyOk

```python
class HandleReplyOk(TypedDict)
```

`HandleReply` member `ok`.

Success: job outcome.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: job outcome.

### HandleReplyErr

```python
class HandleReplyErr(TypedDict)
```

`HandleReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### HealthOk

```python
class HealthOk(TypedDict)
```

Typed liveness report.

**Attributes**

- `ok`: True when the guest is healthy enough for traffic.
- `detail`: Short human status line; empty when none.

### HealthReplyOk

```python
class HealthReplyOk(TypedDict)
```

`HealthReply` member `ok`.

Success: health status.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: health status.

### HealthReplyErr

```python
class HealthReplyErr(TypedDict)
```

`HealthReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### AdapterSessionReplyOk

```python
class AdapterSessionReplyOk(TypedDict)
```

`AdapterSessionReply` member `ok`.

Success: opened `AdapterDatabaseSession` capability.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: opened `AdapterDatabaseSession` capability.

### AdapterSessionReplyErr

```python
class AdapterSessionReplyErr(TypedDict)
```

`AdapterSessionReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### GuestDatabaseReplyOk

```python
class GuestDatabaseReplyOk(TypedDict)
```

`GuestDatabaseReply` member `ok`.

Success: opened `GuestDatabase` capability.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: opened `GuestDatabase` capability.

### GuestDatabaseReplyErr

```python
class GuestDatabaseReplyErr(TypedDict)
```

`GuestDatabaseReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### ByteSource

```python
class ByteSource(Protocol)
```

Transferred readable byte stream. The capability *is* the stream; callers
pull bounded windows. Abort is capability drop / RPC cancel. A failed pull
MUST set `err` — never a successful empty EOF.

#### ByteSource.pull

```python
async def pull(self, max_bytes: int) -> PullReply
```

Pull the next window of bytes. `done = true` on the final chunk.

**Args**

- `max_bytes`: Upper bound for this window; at most `maxStreamWindowBytes`.

**Returns**

`PullReply`

### Destination

```python
class Destination(Protocol)
```

Object store the host writes acquired media into (`output.*` plugins).
Keys are relative object paths; the plugin owns the physical layout.

#### Destination.head

```python
async def head(self, key: str) -> HeadReply
```

Metadata probe. `found = false` is a success, not `not_found`.

**Args**

- `key`: Object key to probe.

**Returns**

`HeadReply`

#### Destination.list

```python
async def list(self, options: ListOptions) -> ListReply
```

Page through objects under a prefix; at most `maxListPage` per page.

**Args**

- `options`: Prefix, cursor, and page size.

**Returns**

`ListReply`

#### Destination.get

```python
async def get(self, key: str, options: ReadOptions) -> GetReply
```

Open an object (or a byte range of it) for reading.

**Args**

- `key`: Object key to read.
- `options`: Optional byte range.

**Returns**

`GetReply`

#### Destination.put

```python
async def put(self, key: str, body: ByteSource, options: WriteOptions) -> PutReply
```

Store an object from a transferred byte stream.

**Args**

- `key`: Object key to write.
- `body`: Streamed object bytes.
- `options`: Content type, length, digest, staging.

**Returns**

`PutReply`

#### Destination.copy

```python
async def copy(self, from_: str, to: str) -> CopyReply
```

Server-side copy (requires `storage.copy`).

**Args**

- `from_`: Source object key.
- `to`: Destination object key.

**Returns**

`CopyReply`

#### Destination.delete

```python
async def delete(self, key: str) -> EmptyReply
```

Remove an object; deleting a missing key is a success.

**Args**

- `key`: Object key to remove.

**Returns**

`EmptyReply`

#### Destination.commit

```python
async def commit(self, key: str, commit_token: str) -> PutReply
```

Finalize a destination-side staged object. Staging itself is `put` with
`stageOnly = true`; bytes must stream into destination-managed temp/multipart
storage, never a complete local spool on host/adapter/broker/guest.

**Args**

- `key`: Object key that was staged.
- `commit_token`: Token from `WriteOptions.commitToken`.

**Returns**

`PutReply`

#### Destination.abort_stage

```python
async def abort_stage(self, key: str, commit_token: str) -> EmptyReply
```

Discard a staged object without publishing it.

**Args**

- `key`: Object key that was staged.
- `commit_token`: Token from `WriteOptions.commitToken`.

**Returns**

`EmptyReply`

### Source

```python
class Source(Protocol)
```

Read-only byte source for job inputs (not a storefront).

#### Source.open

```python
async def open(self, key: str) -> OpenReply
```

Open an object for streaming reads.

**Args**

- `key`: Object key to open.

**Returns**

`OpenReply`

### ProgressSink

```python
class ProgressSink(Protocol)
```

Host-side progress reporter handed to job handlers.

#### ProgressSink.report

```python
async def report(self, percent: float, message: str) -> EmptyReply
```

Report progress; the host coalesces frequent updates.

**Args**

- `percent`: Completion in `[0, 100]`.
- `message`: Short human-readable status line.

**Returns**

`EmptyReply`

### Cancellation

```python
class Cancellation(Protocol)
```

Transport cancellation. SDKs project this into a locally created AbortSignal
(AbortSignal is not a serializable Workers RPC value).

#### Cancellation.poll

```python
async def poll(self) -> bool
```

Non-blocking check; `true` once the host has fenced the invocation.

**Returns**

`bool`

### JobRunner

```python
class JobRunner(Protocol)
```

`[triggers] jobs` handler (`Entrypoints.jobRunner`); runs one durable command.

#### JobRunner.job

```python
async def job(self, controller: JobController) -> HandleReply
```

Run one command invocation to a terminal or suspended outcome. Named
database bindings come from `Bindings.databases` at `open`, not per job.

**Args**

- `controller`: Envelope plus host-served capabilities.

**Returns**

`HandleReply`

### EventConsumer

```python
class EventConsumer(Protocol)
```

`[[events.consumers]]` handler (`Entrypoints.eventConsumer`). Delivery is
at-least-once; consumers must be idempotent on `deduplicationKey`.

#### EventConsumer.event

```python
async def event(self, batch: EventBatch) -> EventBatchReply
```

Deliver one ordered batch of domain events.

**Args**

- `batch`: Events in delivery order.

**Returns**

`EventBatchReply`

### EventPublisher

```python
class EventPublisher(Protocol)
```

`EVENTS` binding: host-served outbox publisher granted through
`Bindings.events`. Event types must be listed in `[[events.producers]]`.

#### EventPublisher.publish

```python
async def publish(self, event: PluginEvent) -> PublishReply
```

Append one event to the outbox (idempotent on `deduplicationKey`).

**Args**

- `event`: Event to publish.

**Returns**

`PublishReply`

### NamedDatabase

```python
class NamedDatabase(TypedDict)
```

One named plugin-owned database binding delivered on `Bindings.databases`.

**Attributes**

- `name`: Binding name from `plugin.toml` `[[databases]]`.
- `database`: Isolated typed SQL session for this binding (plugin-owned schema). Null when the guest does not export this capability.

### QueueConsumer

```python
class QueueConsumer(Protocol)
```

`[[queues.consumers]]` handler (`Entrypoints.queueConsumer`). Delivery is
at-least-once; consumers must be idempotent on `QueueMessage.id`.

#### QueueConsumer.queue

```python
async def queue(self, batch: QueueBatch) -> QueueBatchReply
```

Deliver one ordered batch of messages from one queue.

**Args**

- `batch`: Messages in delivery order.

**Returns**

`QueueBatchReply`

### QueueProducer

```python
class QueueProducer(Protocol)
```

Queue producer binding: host-served send into the queue a
`[[queues.producers]]` row names, granted through `Bindings.queues`.

#### QueueProducer.send

```python
async def send(self, message: QueueSendMessage) -> QueueSendReply
```

Send one message.

**Args**

- `message`: Body, encoding, and delay.

**Returns**

`QueueSendReply`

#### QueueProducer.send_batch

```python
async def send_batch(
    self,
    messages: list[QueueSendMessage],
    delay_seconds: int,
) -> QueueSendReply
```

Send up to `maxListPage` messages in order; `delaySeconds` applies to
every message that carries no delay of its own.

**Args**

- `messages`: Messages in send order.
- `delay_seconds`: Default delay for the batch.

**Returns**

`QueueSendReply`

### NamedQueue

```python
class NamedQueue(TypedDict)
```

One named queue producer delivered on `Bindings.queues`.

**Attributes**

- `name`: Binding name from `plugin.toml` `[[queues.producers]]`.
- `queue`: Queue the binding sends to.
- `producer`: Host-served producer for that queue. Null when the guest does not export this capability.

### KvNamespace

```python
class KvNamespace(Protocol)
```

Durable key/value namespace: a Workers `KVNamespace`-shaped binding the host
serves from the library database, scoped to the plugin and the namespace a
`[[kv_namespaces]]` row names, granted through `Bindings.kv`. Values live in
the cluster, so an instance moved to another host reads its own keys.

#### KvNamespace.get

```python
async def get(self, key: str) -> KvGetReply
```

Read one key; `found = false` when absent or expired.

**Args**

- `key`: Key, at most `maxKvKeyBytes` UTF-8 bytes.

**Returns**

`KvGetReply`

#### KvNamespace.put

```python
async def put(self, key: str, value: bytes, options: KvPutOptions) -> KvPutReply
```

Write one key, replacing any previous value and metadata.

**Args**

- `key`: Key, at most `maxKvKeyBytes` UTF-8 bytes.
- `value`: Value, at most the namespace's `max_value_bytes`.
- `options`: Expiration and metadata.

**Returns**

`KvPutReply`

#### KvNamespace.delete

```python
async def delete(self, key: str) -> KvDeleteReply
```

Delete one key; a missing key is not an error.

**Args**

- `key`: Key to delete.

**Returns**

`KvDeleteReply`

#### KvNamespace.list

```python
async def list(self, options: KvListOptions) -> KvListReply
```

List keys in lexical order, one page at a time.

**Args**

- `options`: Prefix, cursor, and page size.

**Returns**

`KvListReply`

### NamedKv

```python
class NamedKv(TypedDict)
```

One named durable namespace delivered on `Bindings.kv`.

**Attributes**

- `name`: Binding name from `plugin.toml` `[[kv_namespaces]]`.
- `namespace`: Namespace the binding is scoped to.
- `kv`: Host-served namespace. Null when the guest does not export this capability.

### ContentSource

```python
class ContentSource(Protocol)
```

Storefront content source (not byte Source). Every method takes and
returns typed structs from the "Typed method payloads" section.

#### ContentSource.login

```python
async def login(self, params: LoginParams) -> LoginReply
```

Connect an account (password or one-shot OAuth).

**Args**

- `params`: Credentials, marketplace, callback wiring.

**Returns**

`LoginReply`

#### ContentSource.scan

```python
async def scan(self, params: ScanParams) -> ScanReply
```

Sync library rows for one or more accounts.

**Args**

- `params`: Accounts, paging, and host-sealed credentials.

**Returns**

`ScanReply`

#### ContentSource.fetch_title

```python
async def fetch_title(self, params: FetchTitleParams) -> FetchTitleReply
```

Download and decrypt one title into `cacheDir`.

**Args**

- `params`: Title, credentials, and fetch options.

**Returns**

`FetchTitleReply`

#### ContentSource.list_accounts

```python
async def list_accounts(self) -> SourceAccountsReply
```

Enumerate accounts the guest knows about.

**Returns**

`SourceAccountsReply`

#### ContentSource.login_start

```python
async def login_start(self, params: LoginParams) -> LoginStartReply
```

Begin an interactive OAuth login; returns a session id.

**Args**

- `params`: Same shape as `login`; host fills callback IPC.

**Returns**

`LoginStartReply`

#### ContentSource.login_complete

```python
async def login_complete(self, params: LoginCompleteParams) -> LoginReply
```

Finish an interactive OAuth login started by `loginStart`.

**Args**

- `params`: Session id from `loginStart`.

**Returns**

`LoginReply`

#### ContentSource.search_catalog

```python
async def search_catalog(self, params: SearchCatalogParams) -> CatalogHitsReply
```

Free-text storefront catalog search.

**Args**

- `params`: Query, region, paging, sort, facet.

**Returns**

`CatalogHitsReply`

#### ContentSource.expand_candidates

```python
async def expand_candidates(self, params: ExpandCandidatesParams) -> CatalogHitsReply
```

Related-title expansion from a seed title.

**Args**

- `params`: Seed identity fields and limit.

**Returns**

`CatalogHitsReply`

#### ContentSource.purchase_hint

```python
async def purchase_hint(self, params: PurchaseHintParams) -> PurchaseHintReply
```

Purchase link / price hint for one title.

**Args**

- `params`: Identity fields and price flag.

**Returns**

`PurchaseHintReply`

#### ContentSource.list_deals

```python
async def list_deals(self, params: ListDealsParams) -> CatalogHitsReply
```

Current storefront deals.

**Args**

- `params`: Optional result cap.

**Returns**

`CatalogHitsReply`

#### ContentSource.health

```python
async def health(self) -> HealthReply
```

Liveness / readiness probe.

**Returns**

`HealthReply`

#### ContentSource.diagnose

```python
async def diagnose(self) -> DiagnoseReply
```

Human-readable diagnostic lines.

**Returns**

`DiagnoseReply`

#### ContentSource.catalog_detail

```python
async def catalog_detail(self, params: CatalogDetailParams) -> CatalogDetailReply
```

Full catalog record for one product.

**Args**

- `params`: Product id and optional ISBN.

**Returns**

`CatalogDetailReply`

#### ContentSource.revoke_account

```python
async def revoke_account(self, params: RevokeAccountParams) -> EmptyReply
```

Undo store-side state for one account before the host deletes its
stored credentials (for example, deregister the device registered at
login). Best effort: the host revokes the account whatever this returns.

**Args**

- `params`: Account id and its stored credentials.

**Returns**

`EmptyReply`

### RemoteLibrary

```python
class RemoteLibrary(Protocol)
```

`remoteLibrary` entrypoint: long-running remote-library lifecycle (start /
stop, library rescan, listening sync, external-user polling). Event delivery
is `EventConsumer`; credential verification is `Oidc`.

#### RemoteLibrary.health

```python
async def health(self) -> HealthReply
```

Liveness / readiness probe.

**Returns**

`HealthReply`

#### RemoteLibrary.start

```python
async def start(self) -> EmptyReply
```

Start background work after the host has granted bindings.

**Returns**

`EmptyReply`

#### RemoteLibrary.stop

```python
async def stop(self) -> EmptyReply
```

Stop background work; the host may drop the capability afterwards.

**Returns**

`EmptyReply`

#### RemoteLibrary.diagnose

```python
async def diagnose(self) -> DiagnoseReply
```

Human-readable diagnostic lines.

**Returns**

`DiagnoseReply`

#### RemoteLibrary.scan_library

```python
async def scan_library(self, params: ScanLibraryParams) -> EmptyReply
```

Re-sync the remote library.

**Args**

- `params`: Full-rescan flag.

**Returns**

`EmptyReply`

#### RemoteLibrary.sync_listening

```python
async def sync_listening(self) -> SyncListeningReply
```

Push / pull listening progress.

**Returns**

`SyncListeningReply`

#### RemoteLibrary.poll_events

```python
async def poll_events(self) -> EventPollReply
```

Drain external users the remote side observed since the last poll.

**Returns**

`EventPollReply`

### PluginCli

```python
class PluginCli(Protocol)
```

`cli` entrypoint: guest commands under `bookclerk plugins <id> <command>`.

#### PluginCli.describe

```python
async def describe(self) -> CliSchemaReply
```

Declared CLI surface.

**Returns**

`CliSchemaReply`

#### PluginCli.invoke

```python
async def invoke(self, params: CliInvokeParams) -> CliInvokeReply
```

Run one plugin CLI command.

**Args**

- `params`: Command name and argument values.

**Returns**

`CliInvokeReply`

### Oidc

```python
class Oidc(Protocol)
```

`oidc` entrypoint: relying-party client templates and credential
verification on behalf of the host authorization server.

#### Oidc.clients

```python
async def clients(self) -> OidcClientsReply
```

Plugin-provided OIDC AS client templates. Empty list when unused.

**Returns**

`OidcClientsReply`

#### Oidc.authenticate_user

```python
async def authenticate_user(self, params: AuthenticateUserParams) -> ExternalUserReply
```

Verify remote credentials on behalf of the host.

**Args**

- `params`: Username and password.

**Returns**

`ExternalUserReply`

### EventConsumerSpec

```python
class EventConsumerSpec(TypedDict)
```

One declared event consumer: a `[[events.consumers]]` row the default
entrypoint's `event(batch)` handler accepts.

**Attributes**

- `eventType`: Versioned event type (snake_case, e.g. `book_acquired`).
- `schemaVersions`: Schema versions the guest can consume; never empty.
- `supportsSuspend`: Whether `EventResult.suspended` is supported for this type.

### PluginCapabilities

```python
class PluginCapabilities(TypedDict)
```

Typed capability declaration returned by `describe()`. The host compares
it with `plugin.toml` and the operator grant; widening is rejected at spawn.

**Attributes**

- `entrypoints`: Named entrypoints the guest exports.
- `consumes`: Event types the default entrypoint consumes (`event(batch)` trigger).
- `produces`: Event types the guest may publish through its `EVENTS` binding.
- `jobs`: Command types the default entrypoint runs (`job(controller)` trigger).
- `databases`: Plugin-owned database binding names (`[[databases]]`).
- `bindings`: Other named bindings the guest expects on `env` (`CONFIG`, `SECRETS`, `WORK_FS`, `OAUTH`, `KV`, `EVENTS`, `[[queues.producers]]` names, ...).
- `queueConsumers`: Queues the default entrypoint consumes (`queue(batch)` trigger).
- `queueProducers`: Queues the guest may send to through its `[[queues.producers]]` bindings.

### Brand

```python
class Brand(TypedDict)
```

Portal brand crossing the RPC boundary. Distinct from `plugin.toml`
`logo`: `iconUrl` is the live URL or data URI the SPA renders.

**Attributes**

- `id`: Brand id (often matches the plugin id); empty means "no brand".
- `name`: Display name shown next to the brand swatch.
- `bg`: Background CSS color (hex or named).
- `fg`: Foreground CSS color for text on `bg`.
- `accent`: Accent CSS color for highlights / CTAs.
- `iconUrl`: Icon URL or data URI for the portal. Omitted when absent (wire zero value).

### ConfigOption

```python
class ConfigOption(TypedDict)
```

One discoverable config option group advertised for sources.

**Attributes**

- `key`: Config key under the plugin's `config.toml` table.
- `label`: Operator-facing label for the option group.
- `values`: Allowed selectable values for this key.

### ConfigOptionValue

```python
class ConfigOptionValue(TypedDict)
```

One selectable value under a `ConfigOption`.

**Attributes**

- `id`: Value written to config when selected.
- `label`: Operator-facing label for this value.

### CliSchema

```python
class CliSchema(TypedDict)
```

Declared plugin CLI surface (`PluginCli.describe` / `describe().cli`).

**Attributes**

- `commands`: Commands exposed as `bookclerk plugins <id> <command> ...`.

### CliCommandSpec

```python
class CliCommandSpec(TypedDict)
```

One plugin CLI command under `CliSchema`.

**Attributes**

- `name`: Command verb after the plugin id (for example "ping").
- `about`: Short help text for `--help`. Omitted when absent (wire zero value).
- `args`: Argument / flag specs for this command.

### CliArgSpec

```python
class CliArgSpec(TypedDict)
```

One CLI argument or flag under a `CliCommandSpec`.

**Attributes**

- `name`: Internal arg name used as `CliArg.name` on invoke.
- `long`: Long flag without leading dashes (e.g. "message" -> `--message`). Omitted when absent (wire zero value).
- `short`: Short flag character (e.g. "m" -> `-m`). Omitted when absent (wire zero value).
- `kind`: Parsed value kind.
- `required`: When true, the host rejects invoke if the arg is missing.
- `default`: Default string form when the operator omits the arg. Omitted when absent (wire zero value).
- `about`: Help text for this arg. Omitted when absent (wire zero value).
- `positional`: When true, the arg is positional rather than a flagged option.

### CliArg

```python
class CliArg(TypedDict)
```

One named argument value passed to `PluginCli.invoke`.

**Attributes**

- `name`: Arg name matching a `CliArgSpec.name`.
- `value`: String form of the value (the guest parses per `CliArgSpec.kind`).

### CliInvokeParams

```python
class CliInvokeParams(TypedDict)
```

Params of `PluginCli.invoke`.

**Attributes**

- `command`: Command name matching a `CliCommandSpec.name`.
- `args`: Named argument values.

### CliInvokeResult

```python
class CliInvokeResult(TypedDict)
```

Result of `PluginCli.invoke`.

**Attributes**

- `exitCode`: Process-style exit code (0 = success).
- `stdout`: Captured standard output text.
- `stderr`: Captured standard error text.
- `payload`: Structured payload for machine consumers; empty `mediaType` when absent.

### CliSchemaReplyOk

```python
class CliSchemaReplyOk(TypedDict)
```

`CliSchemaReply` member `ok`.

Success: declared CLI surface.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: declared CLI surface.

### CliSchemaReplyErr

```python
class CliSchemaReplyErr(TypedDict)
```

`CliSchemaReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### CliInvokeReplyOk

```python
class CliInvokeReplyOk(TypedDict)
```

`CliInvokeReply` member `ok`.

Success: command output.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: command output.

### CliInvokeReplyErr

```python
class CliInvokeReplyErr(TypedDict)
```

`CliInvokeReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### DatabaseAdapterConfig

```python
class DatabaseAdapterConfig(TypedDict)
```

Author-facing database adapter configuration carried in
`DatabaseContext.adapter`. This is the generic bootstrap mechanism for
third-party adapters: the operator's granted `[database.<id>]` table plus
the scoped writable data dir. First-party host-managed adapters receive
host-private connect params in `DatabaseContext.config` instead.

**Attributes**

- `pluginDataDir`: Scoped writable directory for this plugin (`.../plugins/<id>/data`).
- `settings`: Granted plugin settings (operator `[database.<id>]` table) as `application/json`; `{}` when the operator configured nothing.
- `binding`: Named plugin database binding this open serves; empty for the primary library open. Adapters advertising `DbCapabilities.pluginDatabases` must serve each binding from its own isolated database. Omitted when absent (wire zero value).
- `instanceId`: Host-issued opaque instance id for this (owner plugin, binding) pair. Collision-resistant and stable across re-opens. Empty for the primary library open. Third-party adapters must key isolated databases on this value rather than `binding` alone (two plugins may both declare `DB`). Omitted when absent (wire zero value).
- `openExisting`: When true, open an existing binding unit and do not provision a missing one (read-only backup capture). False lets the adapter create the unit.

### DiagnoseResult

```python
class DiagnoseResult(TypedDict)
```

Operator-facing diagnostic lines printed by `bookclerk plugins diagnose`.

**Attributes**

- `lines`: Human-readable probe lines.

### DiagnoseReplyOk

```python
class DiagnoseReplyOk(TypedDict)
```

`DiagnoseReply` member `ok`.

Success: diagnostic lines.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: diagnostic lines.

### DiagnoseReplyErr

```python
class DiagnoseReplyErr(TypedDict)
```

`DiagnoseReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### SourceAccount

```python
class SourceAccount(TypedDict)
```

Source account metadata returned from login and stored by the host.

**Attributes**

- `accountId`: Stable account id within this source plugin.
- `source`: Source plugin id (the host forces this to the guest's install id).
- `marketplace`: Storefront marketplace / region code (for example `us`, `uk`).
- `label`: Operator-facing label. Omitted when absent (wire zero value).
- `scanEnabled`: When true, bare / scheduled scans include this account. Explicit CLI `--account` bypasses this flag.

### LoginParams

```python
class LoginParams(TypedDict)
```

Params of `ContentSource.login` and `ContentSource.loginStart`. Password
sources fill email/password; OAuth sources use callback / external fields.
There is no files-dir root or library DB path -- only `pluginDataDir`.

**Attributes**

- `pluginDataDir`: Scoped writable directory for this plugin only (`.../plugins/<id>/data`).
- `marketplace`: Marketplace / locale for the storefront; empty means the guest default.
- `label`: Operator label stored on the account row. Omitted when absent (wire zero value).
- `email`: Account email / username for password logins; empty for pure OAuth. Omitted when absent (wire zero value).
- `password`: Account password for password logins; never logged; empty for OAuth. Omitted when absent (wire zero value).
- `force`: When true, overwrite an existing sealed credential for this account.
- `callbackBind`: Bind address for OAuth callback servers (`host:port`). Ignored when `callbackIpc` is set (host owns the TCP listener). Omitted when absent (wire zero value).
- `callbackIpc`: Host-owned callback IPC endpoint the guest must connect to. When set (with `callbackPublicBase`), the guest must not bind a TCP listener. Omitted when absent (wire zero value).
- `callbackPublicBase`: Public base URL for the host TCP listener, e.g. `http://127.0.0.1:12345`. Omitted when absent (wire zero value).
- `external`: When true, use external / paste-redirect OAuth instead of a local callback server.
- `responseUrl`: Pre-supplied OAuth redirect URL (paste flow). Omitted when absent (wire zero value).
- `showQr`: Prefer QR output when the guest supports it.
- `timeoutSecs`: Seconds to wait for OAuth callback capture; guest default when 0. Omitted when absent (wire zero value).
- `extra`: Store-specific knobs as `application/json`; guests may ignore unknowns.

### LoginResult

```python
class LoginResult(TypedDict)
```

Result of `ContentSource.login` / `loginComplete`: account metadata plus
opaque credentials for the host to seal into `encrypted_secrets`
(`provider = plugin id`). Guests never write secrets into the library DB.

**Attributes**

- `account`: Account row fields for the host to upsert.
- `credentials`: Opaque credential blob the host seals; empty when login only refreshed metadata. Guests choose the encoding (typically JSON bytes). Omitted when absent (wire zero value).

### LoginStartResult

```python
class LoginStartResult(TypedDict)
```

Result of `ContentSource.loginStart` (interactive OAuth). The operator
opens `url`; `loginComplete` later uses `sessionId`.

**Attributes**

- `sessionId`: Opaque session id for `loginComplete`.
- `url`: Browser URL the operator should open to complete OAuth.

### LoginCompleteParams

```python
class LoginCompleteParams(TypedDict)
```

Params of `ContentSource.loginComplete`.

**Attributes**

- `sessionId`: Session id previously returned by `loginStart`.

### LoginReplyOk

```python
class LoginReplyOk(TypedDict)
```

`LoginReply` member `ok`.

Success: account plus credentials to seal.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: account plus credentials to seal.

### LoginReplyErr

```python
class LoginReplyErr(TypedDict)
```

`LoginReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### LoginStartReplyOk

```python
class LoginStartReplyOk(TypedDict)
```

`LoginStartReply` member `ok`.

Success: session id and browser URL.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: session id and browser URL.

### LoginStartReplyErr

```python
class LoginStartReplyErr(TypedDict)
```

`LoginStartReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### AccountCredential

```python
class AccountCredential(TypedDict)
```

Host-sealed credentials for one account, delivered on `scan`.

**Attributes**

- `accountId`: Account id the blob belongs to.
- `credentials`: Opaque credential bytes exactly as the guest returned them at login.

### ScanParams

```python
class ScanParams(TypedDict)
```

Params of `ContentSource.scan`. The host injects sealed credentials so the
plugin does not need a private credential store under `pluginDataDir`.

**Attributes**

- `pluginDataDir`: Scoped plugin data directory.
- `accounts`: Account ids to scan; empty means all scan-enabled accounts.
- `pageSize`: Storefront page size; the host always sends an explicit value.
- `importEpisodes`: When true, import podcast/episode-style rows.
- `importPlusTitles`: When true, import Plus/catalog entitlement titles.
- `credentials`: Host-loaded credential blobs for the requested accounts.

### ScanBook

```python
class ScanBook(TypedDict)
```

One library title returned by `ContentSource.scan`. The host upserts these
rows and forces `source` to the plugin id.

**Attributes**

- `accountId`: Account that owns this library entry.
- `productId`: Storefront product / SKU id.
- `title`: Primary title string.
- `marketplace`: Marketplace / region when known. Omitted when absent (wire zero value).
- `asin`: Amazon ASIN when the storefront exposes one. Omitted when absent (wire zero value).
- `isbn`: ISBN when the storefront exposes one. Omitted when absent (wire zero value).
- `authors`: Comma- or guest-formatted author list. Omitted when absent (wire zero value).
- `narrators`: Comma- or guest-formatted narrator list. Omitted when absent (wire zero value).
- `series`: Series name when applicable. Omitted when absent (wire zero value).
- `seriesIndex`: Series index / sequence label. Omitted when absent (wire zero value).
- `contentKind`: Content classification (e.g. `book` vs `episode`). Omitted when absent (wire zero value).
- `publisher`: Publisher name when known. Omitted when absent (wire zero value).
- `lengthMinutes`: Runtime in whole minutes when known. Omitted when absent (wire zero value).
- `subtitle`: Subtitle when distinct from `title`. Omitted when absent (wire zero value).

### ScanSummary

```python
class ScanSummary(TypedDict)
```

Summary result of `ContentSource.scan`.

**Attributes**

- `accounts`: Number of accounts touched during the scan.
- `booksUpserted`: Count of titles the guest expects the host to upsert; may mirror `books.length`.
- `pages`: Number of storefront pages fetched.
- `skippedDisabled`: Accounts skipped because `scanEnabled` was false.
- `books`: Titles for the host to upsert. Prefer this over plugin-side DB writes.

### ScanReplyOk

```python
class ScanReplyOk(TypedDict)
```

`ScanReply` member `ok`.

Success: scan summary and titles to upsert.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: scan summary and titles to upsert.

### ScanReplyErr

```python
class ScanReplyErr(TypedDict)
```

`ScanReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### FetchOptions

```python
class FetchOptions(TypedDict)
```

Fetch-relevant acquire knobs the host forwards so external load matches
in-process. Packaging / naming knobs stay host-side.

**Attributes**

- `widevine`: Prefer Widevine/CENC download when the store offers it.
- `xheAac`: Prefer xHE-AAC on the Widevine path when offered.
- `widevineCdmPath`: Local Widevine `.wvd` path granted to the guest. Omitted when absent (wire zero value).
- `widevineCdmProvider`: Remote L3 CDM provider URL; empty means the classic default, `off` disables remote provisioning. Omitted when absent (wire zero value).
- `downloadCover`: When true, download a cover image alongside audio.
- `downloadPdf`: When true, download a companion PDF when the store exposes one.
- `coverSize`: Cover image size request (`500`, `1215`, or `native`).
- `chapterLayout`: Preferred chapter API layout when fetching (`tree` or `flat`).
- `stripAudibleBrandAudio`: When true, trim Audible brand intro/outro from the remux window.
- `downloadClipsBookmarks`: When true, download clips/bookmarks sidecars when offered.
- `retainAaxFile`: When true, keep the encrypted download in storage.
- `downloadSpeedLimitKbps`: Fetch speed cap in KB/s (`0` = unlimited).
- `saveMetadataJson`: When true, persist raw catalog API JSON as `metadata.json`.

### FetchTitleParams

```python
class FetchTitleParams(TypedDict)
```

Params of `ContentSource.fetchTitle`. The plugin writes media under
`cacheDir` and returns plain (DRM-free) paths. The host injects
credentials; guests must not open `library.db` or `master.key`.

**Attributes**

- `pluginDataDir`: Scoped plugin data directory.
- `accountId`: Account whose credentials apply.
- `titleId`: Library / storefront title id to download.
- `cacheDir`: Absolute path the guest should write media into (jail-granted TMPDIR).
- `credentials`: Host-loaded credential blob for this account; empty when unavailable. Omitted when absent (wire zero value).
- `sourceConfig`: Granted `[sources.<id>]` table as `application/json`.
- `fetch`: Fetch-relevant acquire options.

### PlainPart

```python
class PlainPart(TypedDict)
```

One plain audio part written under the cache directory.

**Attributes**

- `path`: Absolute path to the part file under `cacheDir`.
- `title`: Part title (disc / chapter label). Omitted when absent (wire zero value).
- `durationMs`: Duration of this part in milliseconds when known. Omitted when absent (wire zero value).

### ChapterMarker

```python
class ChapterMarker(TypedDict)
```

One chapter marker of a fetched title.

**Attributes**

- `title`: Chapter title.
- `startMs`: Chapter start offset in milliseconds from the beginning of the title.

### PlainFetch

```python
class PlainFetch(TypedDict)
```

Plain (DRM-free) fetch result. Sources always return decrypted media; DRM
guests decrypt before responding.

**Attributes**

- `parts`: Ordered audio part files written under the cache directory.
- `m4bPath`: Single M4B path when the guest assembled one. Omitted when absent (wire zero value).
- `coverPath`: Cover image path under the cache directory. Omitted when absent (wire zero value).
- `chapters`: Chapter markers; empty when unknown.
- `pdfUrl`: Companion PDF download URL when the store exposes one. Omitted when absent (wire zero value).

### FetchTitleReplyOk

```python
class FetchTitleReplyOk(TypedDict)
```

`FetchTitleReply` member `ok`.

Success: plain media paths.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: plain media paths.

### FetchTitleReplyErr

```python
class FetchTitleReplyErr(TypedDict)
```

`FetchTitleReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### SourceAccounts

```python
class SourceAccounts(TypedDict)
```

Success payload of `ContentSource.listAccounts`.

**Attributes**

- `accounts`: Accounts the guest knows about.

### SourceAccountsReplyOk

```python
class SourceAccountsReplyOk(TypedDict)
```

`SourceAccountsReply` member `ok`.

Success: account list.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: account list.

### SourceAccountsReplyErr

```python
class SourceAccountsReplyErr(TypedDict)
```

`SourceAccountsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### SearchCatalogParams

```python
class SearchCatalogParams(TypedDict)
```

Params of `ContentSource.searchCatalog`.

**Attributes**

- `query`: Free-text search query.
- `region`: Storefront region / marketplace code; empty means the guest default.
- `limit`: Maximum hits to return; the host always sends an explicit value.
- `page`: 1-based page for storefronts that page.
- `sort`: Sort order.
- `field`: Facet restriction.
- `language`: Preferred content language (soft-prioritize; e.g. `en`). Omitted when absent (wire zero value).

### ExpandCandidatesParams

```python
class ExpandCandidatesParams(TypedDict)
```

Params of `ContentSource.expandCandidates`. Seed fields identify a known
title; the guest returns related catalog hits.

**Attributes**

- `source`: Source plugin id hint when expanding across storefronts.
- `productId`: Seed storefront product id.
- `title`: Seed title text.
- `authors`: Seed authors string. Omitted when absent (wire zero value).
- `narrators`: Seed narrators string. Omitted when absent (wire zero value).
- `series`: Seed series name. Omitted when absent (wire zero value).
- `seriesAsin`: Seed series ASIN when known. Omitted when absent (wire zero value).
- `asin`: Seed Amazon ASIN. Omitted when absent (wire zero value).
- `isbn`: Seed ISBN. Omitted when absent (wire zero value).
- `region`: Storefront region / marketplace code.
- `limit`: Maximum candidates to return; the host always sends an explicit value.

### PurchaseHintParams

```python
class PurchaseHintParams(TypedDict)
```

Params of `ContentSource.purchaseHint`. At least one identity field
(`productId` / `asin` / `isbn` / title+authors) should be set; guests may
return `invalid_params` when none are usable.

**Attributes**

- `productId`: Storefront product id when known. Omitted when absent (wire zero value).
- `title`: Title text for fuzzy lookup. Omitted when absent (wire zero value).
- `authors`: Authors string for fuzzy lookup. Omitted when absent (wire zero value).
- `asin`: Amazon ASIN when known. Omitted when absent (wire zero value).
- `isbn`: ISBN when known. Omitted when absent (wire zero value).
- `region`: Storefront region / marketplace code.
- `withPrice`: When true, guests should include live price fields when available.

### ListDealsParams

```python
class ListDealsParams(TypedDict)
```

Params of `ContentSource.listDeals`.

**Attributes**

- `limit`: Maximum number of deals to return; guest default when 0. Omitted when absent (wire zero value).

### CatalogDetailParams

```python
class CatalogDetailParams(TypedDict)
```

Params of `ContentSource.catalogDetail`.

**Attributes**

- `productId`: Store product id (Libro ISBN or ISBN-slug).
- `isbn`: ISBN when it differs from `productId`. Omitted when absent (wire zero value).

### RevokeAccountParams

```python
class RevokeAccountParams(TypedDict)
```

Params of `ContentSource.revokeAccount`. The host sends the credentials it
is about to delete so the guest can sign the device out at the store.

**Attributes**

- `accountId`: Account being revoked.
- `credentials`: Decrypted credential blob for this account, exactly as login returned it; never log it. Empty when none is stored. Omitted when absent (wire zero value).

### CatalogHit

```python
class CatalogHit(TypedDict)
```

One catalog / candidate hit returned by `searchCatalog`,
`expandCandidates`, `listDeals`, and `catalogDetail`.

**Attributes**

- `productId`: Storefront product / SKU id.
- `title`: Primary title.
- `authors`: Authors string when known. Omitted when absent (wire zero value).
- `narrators`: Narrators string when known. Omitted when absent (wire zero value).
- `series`: Series name when applicable. Omitted when absent (wire zero value).
- `seriesIndex`: Series index / sequence label. Omitted when absent (wire zero value).
- `asin`: Amazon ASIN when known. Omitted when absent (wire zero value).
- `isbn`: ISBN when known. Omitted when absent (wire zero value).
- `url`: Storefront product page URL. Omitted when absent (wire zero value).
- `coverUrl`: Cover image URL. Omitted when absent (wire zero value).
- `origin`: Hit origin label (plugin id or storefront name).
- `subtitle`: Subtitle when distinct from `title`. Omitted when absent (wire zero value).
- `description`: Long description / blurb when fetched. Omitted when absent (wire zero value).
- `publisher`: Publisher name when known. Omitted when absent (wire zero value).
- `lengthMinutes`: Runtime in whole minutes. Omitted when absent (wire zero value).
- `publishedAt`: Publication date string as provided by the storefront. Omitted when absent (wire zero value).
- `categories`: Category / genre labels as a single string when known. Omitted when absent (wire zero value).
- `language`: Content language code when known. Omitted when absent (wire zero value).
- `priceCents`: Current price in minor units (cents). Omitted when absent (wire zero value).
- `currency`: ISO currency code for `priceCents`. Omitted when absent (wire zero value).
- `priceLabel`: Pre-formatted price for display. Omitted when absent (wire zero value).
- `ratingOverall`: Aggregate rating when known. Omitted when absent (wire zero value).
- `ratingCount`: Number of ratings when known. Omitted when absent (wire zero value).
- `abridgement`: Whether the edition is abridged when the storefront says so.

### CatalogHits

```python
class CatalogHits(TypedDict)
```

Success payload of the catalog list methods.

**Attributes**

- `hits`: Hits in storefront order.

### CatalogHitsReplyOk

```python
class CatalogHitsReplyOk(TypedDict)
```

`CatalogHitsReply` member `ok`.

Success: catalog hits.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: catalog hits.

### CatalogHitsReplyErr

```python
class CatalogHitsReplyErr(TypedDict)
```

`CatalogHitsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### CatalogDetail

```python
class CatalogDetail(TypedDict)
```

Success payload of `ContentSource.catalogDetail`.

**Attributes**

- `found`: False when the product is unknown to the storefront (`hit` is empty).
- `hit`: Full catalog record when `found`.

### CatalogDetailReplyOk

```python
class CatalogDetailReplyOk(TypedDict)
```

`CatalogDetailReply` member `ok`.

Success: detail record or not-found marker.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: detail record or not-found marker.

### CatalogDetailReplyErr

```python
class CatalogDetailReplyErr(TypedDict)
```

`CatalogDetailReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### PurchaseHint

```python
class PurchaseHint(TypedDict)
```

Purchase hint for SPA / CLI purchase deep-links.

**Attributes**

- `productId`: Storefront product id.
- `title`: Title when resolved. Omitted when absent (wire zero value).
- `url`: Purchase or product-page URL. Omitted when absent (wire zero value).
- `priceCents`: Current price in minor units. Omitted when absent (wire zero value).
- `currency`: ISO currency code for price fields. Omitted when absent (wire zero value).
- `priceLabel`: Pre-formatted current price. Omitted when absent (wire zero value).
- `listPriceCents`: List / MSRP price in minor units. Omitted when absent (wire zero value).
- `listPriceLabel`: Pre-formatted list price. Omitted when absent (wire zero value).
- `memberPriceCents`: Member / Plus price in minor units. Omitted when absent (wire zero value).
- `memberPriceLabel`: Pre-formatted member price. Omitted when absent (wire zero value).

### PurchaseHintResult

```python
class PurchaseHintResult(TypedDict)
```

Success payload of `ContentSource.purchaseHint`.

**Attributes**

- `found`: False when the guest could not resolve the title (`hint` is empty).
- `hint`: Purchase hint when `found`.

### PurchaseHintReplyOk

```python
class PurchaseHintReplyOk(TypedDict)
```

`PurchaseHintReply` member `ok`.

Success: hint or not-found marker.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: hint or not-found marker.

### PurchaseHintReplyErr

```python
class PurchaseHintReplyErr(TypedDict)
```

`PurchaseHintReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### ScanLibraryParams

```python
class ScanLibraryParams(TypedDict)
```

Params of `RemoteLibrary.scanLibrary` (remote library sync).

**Attributes**

- `force`: When true, force a full rescan even if the guest would otherwise incremental-sync.

### AuthenticateUserParams

```python
class AuthenticateUserParams(TypedDict)
```

Params of `Oidc.authenticateUser`.

**Attributes**

- `username`: Integration username / login id.
- `password`: Integration password; never logged by the host.

### ExternalUser

```python
class ExternalUser(TypedDict)
```

One external user observed by an integration. The host may mint claim
tickets without exposing portal details to the guest.

**Attributes**

- `provider`: Integration provider id (often the plugin id).
- `externalUserId`: Provider-scoped user id.
- `displayName`: Display name for UI. Omitted when absent (wire zero value).
- `accessToken`: Ephemeral remote token (e.g. ABS JWT). Guest-to-host only; never persisted. Omitted when absent (wire zero value).

### ExternalUserReplyOk

```python
class ExternalUserReplyOk(TypedDict)
```

`ExternalUserReply` member `ok`.

Success: verified external user.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: verified external user.

### ExternalUserReplyErr

```python
class ExternalUserReplyErr(TypedDict)
```

`ExternalUserReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### EventPollResult

```python
class EventPollResult(TypedDict)
```

Success payload of `RemoteLibrary.pollEvents`: signals for the host to kick
off workflows.

**Attributes**

- `users`: Newly observed external users since the last poll.

### EventPollReplyOk

```python
class EventPollReplyOk(TypedDict)
```

`EventPollReply` member `ok`.

Success: observed users.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: observed users.

### EventPollReplyErr

```python
class EventPollReplyErr(TypedDict)
```

`EventPollReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### ListeningProgress

```python
class ListeningProgress(TypedDict)
```

One listening-progress row. The host upserts into `listening_progress`
tagged with the plugin id; plugins never open the library DB.

**Attributes**

- `externalUserId`: Provider-scoped user id.
- `externalItemId`: Provider-scoped item / library id.
- `identityId`: Bookclerk identity row id when already linked. Omitted when absent (wire zero value).
- `title`: Title text when known. Omitted when absent (wire zero value).
- `authors`: Authors string when known. Omitted when absent (wire zero value).
- `asin`: Amazon ASIN when known. Omitted when absent (wire zero value).
- `isbn`: ISBN when known. Omitted when absent (wire zero value).
- `progress`: Fractional progress in `0.0..=1.0` when the provider reports it. Omitted when absent (wire zero value).
- `currentTimeSeconds`: Current playback position in seconds. Omitted when absent (wire zero value).
- `durationSeconds`: Total duration in seconds when known. Omitted when absent (wire zero value).
- `isFinished`: When true, the provider marks the item finished.
- `lastListenedAtUnixMs`: Last listen timestamp as unix milliseconds (UTC). Omitted when absent (wire zero value).

### SyncListeningResult

```python
class SyncListeningResult(TypedDict)
```

Success payload of `RemoteLibrary.syncListening`.

**Attributes**

- `items`: Progress snapshots to upsert.

### SyncListeningReplyOk

```python
class SyncListeningReplyOk(TypedDict)
```

`SyncListeningReply` member `ok`.

Success: progress snapshots.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: progress snapshots.

### SyncListeningReplyErr

```python
class SyncListeningReplyErr(TypedDict)
```

`SyncListeningReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### PluginEvent

```python
class PluginEvent(TypedDict)
```

Guest-published domain event (`EventPublisher.publish`). The host stamps
`eventId`, `source` (the plugin id), and `accountId` from the invocation;
guests cannot forge either.

**Attributes**

- `eventType`: Snake_case event type; must be listed in `[[events.producers]]`.
- `schemaVersion`: Schema version of `payload`, owned by the event type.
- `deduplicationKey`: Producer idempotency key, unique per (account, source, eventType) in the outbox; a repeat returns `PublishOk.duplicate = true` with the earlier id. Empty publishes unconditionally.
- `payload`: Encoded event payload; at most `maxEventPayloadBytes`.
- `occurredAtUnixMs`: When the producer observed the fact; zero means "now" on the host clock.
- `correlationId`: Trace correlation id; empty inherits `Invocation.correlationId`.
- `causationId`: Id of the event or command that caused this one; empty inherits `Invocation.causationId`.

### PublishOk

```python
class PublishOk(TypedDict)
```

Success payload of `EventPublisher.publish`.

**Attributes**

- `eventId`: Outbox event id (new or the earlier row when `duplicate`).
- `duplicate`: True when `deduplicationKey` matched an existing outbox row.

### PublishReplyOk

```python
class PublishReplyOk(TypedDict)
```

`PublishReply` member `ok`.

Success: outbox row identity.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: outbox row identity.

### PublishReplyErr

```python
class PublishReplyErr(TypedDict)
```

`PublishReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### Invocation

```python
class Invocation(TypedDict)
```

Identity of one `PluginWorker.open` invocation. The host issues ids; guests
echo `correlationId` / `causationId` onto published events.

**Attributes**

- `id`: Unique host-issued invocation id.
- `accountId`: Account scope; empty for operator / host-wide invocations.
- `deadlineUnixMs`: UTC Unix milliseconds; zero when the invocation has no deadline. The host fence is authoritative.
- `correlationId`: Trace correlation id; empty when none.
- `causationId`: Id of the event or command that caused this invocation; empty when none.

### QueueMessage

```python
class QueueMessage(TypedDict)
```

One message on a Workers-style queue, delivered by `QueueConsumer.queue`.

**Attributes**

- `id`: Outbox row id; stable across redeliveries.
- `timestampUnixMs`: When the producer sent the message (UTC Unix milliseconds).
- `body`: Encoded body; at most `maxEventPayloadBytes`.
- `contentType`: Body encoding: `json`, `text`, or `bytes`.
- `attempts`: Delivery counter, starting at 1.
- `source`: Producer plugin id; empty when the host sent it.

### QueueBatch

```python
class QueueBatch(TypedDict)
```

One `QueueConsumer.queue` delivery: messages from one queue in send order.

**Attributes**

- `queue`: Queue the messages were sent to.
- `messages`: Messages in delivery order; the reply carries one `QueueResult` per entry.

### QueueRetry

```python
class QueueRetry(TypedDict)
```

Redeliver a queue message later.

**Attributes**

- `delaySeconds`: Seconds before redelivery; zero applies the consumer's `retry_delay`.
- `reason`: Short human reason.

### QueueResultAck

```python
class QueueResultAck(TypedDict)
```

`QueueResult` member `ack`.

Handled; the host marks the message delivered.

**Attributes**

- `kind`: Always `"ack"`.

### QueueResultRetry

```python
class QueueResultRetry(TypedDict)
```

`QueueResult` member `retry`.

Redeliver later.

**Attributes**

- `kind`: Always `"retry"`.
- `value`: Redeliver later.

### QueueBatchOk

```python
class QueueBatchOk(TypedDict)
```

Success payload of `QueueConsumer.queue`: exactly one entry per
`QueueBatch.messages` entry, in order. A short list is a host-side error
and the missing tail is retried.

**Attributes**

- `results`: Per-message outcomes in delivery order.

### QueueBatchReplyOk

```python
class QueueBatchReplyOk(TypedDict)
```

`QueueBatchReply` member `ok`.

Success: per-message outcomes.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: per-message outcomes.

### QueueBatchReplyErr

```python
class QueueBatchReplyErr(TypedDict)
```

`QueueBatchReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### QueueSendMessage

```python
class QueueSendMessage(TypedDict)
```

One message to send through a `[[queues.producers]]` binding.

**Attributes**

- `body`: Encoded body; at most `maxEventPayloadBytes`.
- `contentType`: Body encoding: `json` (default when empty), `text`, or `bytes`.
- `delaySeconds`: Seconds before the message may be delivered; zero means now.

### QueueSendOk

```python
class QueueSendOk(TypedDict)
```

Success payload of `QueueProducer.send` / `sendBatch`.

**Attributes**

- `messageIds`: Outbox row ids in send order.

### QueueSendReplyOk

```python
class QueueSendReplyOk(TypedDict)
```

`QueueSendReply` member `ok`.

Success: the ids the host assigned.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: the ids the host assigned.

### QueueSendReplyErr

```python
class QueueSendReplyErr(TypedDict)
```

`QueueSendReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### KvPutOptions

```python
class KvPutOptions(TypedDict)
```

Expiration and metadata for `KvNamespace.put`.

**Attributes**

- `expirationUnixMs`: Absolute expiry (UTC Unix milliseconds); zero means none.
- `expirationTtlSeconds`: Relative expiry in seconds from the write; zero means none. Ignored when `expirationUnixMs` is set.
- `metadata`: Opaque JSON metadata text, at most `maxKvMetadataBytes`; empty means none.

### KvGetOk

```python
class KvGetOk(TypedDict)
```

Success payload of `KvNamespace.get`.

**Attributes**

- `found`: True when the key exists and has not expired.
- `value`: Stored value; empty when not found.
- `metadata`: Stored metadata JSON text; empty when none.
- `expirationUnixMs`: Absolute expiry (UTC Unix milliseconds); zero means none.

### KvGetReplyOk

```python
class KvGetReplyOk(TypedDict)
```

`KvGetReply` member `ok`.

Success: the value, when found.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: the value, when found.

### KvGetReplyErr

```python
class KvGetReplyErr(TypedDict)
```

`KvGetReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### KvPutOk

```python
class KvPutOk(TypedDict)
```

Success payload of `KvNamespace.put`.

**Attributes**

- `expirationUnixMs`: Effective absolute expiry (UTC Unix milliseconds); zero means none.

### KvDeleteOk

```python
class KvDeleteOk(TypedDict)
```

Success payload of `KvNamespace.delete`.

**Attributes**

- `existed`: True when a live value existed before the delete.

### KvPutReplyOk

```python
class KvPutReplyOk(TypedDict)
```

`KvPutReply` member `ok`.

Success: the write is durable.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: the write is durable.

### KvPutReplyErr

```python
class KvPutReplyErr(TypedDict)
```

`KvPutReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### KvDeleteReplyOk

```python
class KvDeleteReplyOk(TypedDict)
```

`KvDeleteReply` member `ok`.

Success: the key is gone (or never existed).

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: the key is gone (or never existed).

### KvDeleteReplyErr

```python
class KvDeleteReplyErr(TypedDict)
```

`KvDeleteReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### KvListOptions

```python
class KvListOptions(TypedDict)
```

Page request for `KvNamespace.list`.

**Attributes**

- `prefix`: Only keys starting with this prefix; empty means all.
- `cursor`: Opaque cursor from a previous page; empty means the first page.
- `limit`: Most keys to return; zero means the host default, at most `maxListPage`.

### KvListKey

```python
class KvListKey(TypedDict)
```

One key on a `KvNamespace.list` page.

**Attributes**

- `key`: Key name.
- `metadata`: Stored metadata JSON text; empty when none.
- `expirationUnixMs`: Absolute expiry (UTC Unix milliseconds); zero means none.

### KvListOk

```python
class KvListOk(TypedDict)
```

Success payload of `KvNamespace.list`.

**Attributes**

- `keys`: Keys in lexical order.
- `listComplete`: True when no further page exists.
- `cursor`: Cursor for the next page; empty when `listComplete`.

### KvListReplyOk

```python
class KvListReplyOk(TypedDict)
```

`KvListReply` member `ok`.

Success: one page of keys.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: one page of keys.

### KvListReplyErr

```python
class KvListReplyErr(TypedDict)
```

`KvListReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### DbValueNull

```python
class DbValueNull(TypedDict)
```

`DbValue` member `null`.

SQL NULL with its declared type.

**Attributes**

- `kind`: Always `"null"`.
- `value`: SQL NULL with its declared type.

### DbValueBoolean

```python
class DbValueBoolean(TypedDict)
```

`DbValue` member `boolean`.

Boolean.

**Attributes**

- `kind`: Always `"boolean"`.
- `value`: Boolean.

### DbValueInt64

```python
class DbValueInt64(TypedDict)
```

`DbValue` member `int64`.

Signed 64-bit integer.

**Attributes**

- `kind`: Always `"int64"`.
- `value`: Signed 64-bit integer.

### DbValueFloat64

```python
class DbValueFloat64(TypedDict)
```

`DbValue` member `float64`.

IEEE-754 double.

**Attributes**

- `kind`: Always `"float64"`.
- `value`: IEEE-754 double.

### DbValueText

```python
class DbValueText(TypedDict)
```

`DbValue` member `text`.

UTF-8 text.

**Attributes**

- `kind`: Always `"text"`.
- `value`: UTF-8 text.

### DbValueBytes

```python
class DbValueBytes(TypedDict)
```

`DbValue` member `bytes`.

Raw bytes.

**Attributes**

- `kind`: Always `"bytes"`.
- `value`: Raw bytes.

### DbColumn

```python
class DbColumn(TypedDict)
```

Result column descriptor.

**Attributes**

- `name`: Column name as projected.
- `dbType`: Declared or inferred column type.

### DbRow

```python
class DbRow(TypedDict)
```

One result row.

**Attributes**

- `values`: Cells in `columns` order.

### DbStatement

```python
class DbStatement(TypedDict)
```

One guest statement in an `ExecuteRequest`.

**Attributes**

- `sql`: BookclerkSQL text; at most `maxScalarBytes`.
- `parameters`: Positional bind values.
- `kind`: Statement classification.
- `maxRows`: Row cap for queries; `0` means adapter default.
- `resultSelection`: Which outcome parts to return.

### ExecuteRequest

```python
class ExecuteRequest(TypedDict)
```

Guest statement batch for `GuestDatabase.execute`; runs atomically.

**Attributes**

- `operationId`: Caller-chosen idempotency key.
- `requestHash`: SHA-256 hex of the idempotency-relevant request; empty when omitted.
- `statements`: Statements in execution order; at most `maxStatements`.
- `deadlineUnixMs`: Deadline hint; `0` means none.

### SqlSpan

```python
class SqlSpan(TypedDict)
```

Byte span in the exact canonical SQL string a proof is bound to.

**Attributes**

- `start`: Inclusive start byte offset.
- `end`: Exclusive end byte offset.

### TextCollateSite

```python
class TextCollateSite(TypedDict)
```

One TEXT expression the adapter must collate bytewise (`COLLATE "C"`).

**Attributes**

- `span`: Identifier or string-literal span in canonical SQL.

### IntegerArithSite

```python
class IntegerArithSite(TypedDict)
```

One INTEGER arithmetic expression that must not wrap or error on overflow.

**Attributes**

- `full`: Full expression span (`a + b` or `abs(n)`).
- `lhs`: Left operand (or `abs` argument).
- `rhs`: Right operand (`abs` repeats `lhs`).
- `kind`: Operator.

### PhysicalAccess

```python
class PhysicalAccess(TypedDict)
```

Physical table/column access used for authorization.

**Attributes**

- `table`: Physical table name.
- `column`: Empty = table presence only; "*" = projection wildcard.

### ResolvedAssignment

```python
class ResolvedAssignment(TypedDict)
```

Destination assignment `lhs = rhs` (INSERT, UPDATE, ...).

**Attributes**

- `table`: Target table.
- `column`: Target column.
- `dest`: Declared column type.
- `source`: Resolved type of the assigned expression.

### NamedSqlType

```python
class NamedSqlType(TypedDict)
```

Column name paired with its resolved type.

**Attributes**

- `name`: Column name.
- `sqlType`: Resolved type.

### ColumnReference

```python
class ColumnReference(TypedDict)
```

`REFERENCES` target of one column.

**Attributes**

- `refTable`: Referenced table.
- `refColumns`: Referenced columns.

### OptionalColumnReferenceNone

```python
class OptionalColumnReferenceNone(TypedDict)
```

`OptionalColumnReference` member `none`.

Column has no reference.

**Attributes**

- `kind`: Always `"none"`.

### OptionalColumnReferenceSome

```python
class OptionalColumnReferenceSome(TypedDict)
```

`OptionalColumnReference` member `some`.

Column references another table.

**Attributes**

- `kind`: Always `"some"`.
- `value`: Column references another table.

### ForeignKeyConstraint

```python
class ForeignKeyConstraint(TypedDict)
```

Table-level `FOREIGN KEY` constraint.

**Attributes**

- `columns`: Local columns.
- `refTable`: Referenced table.
- `refColumns`: Referenced columns.

### TableConstraintPrimaryKey

```python
class TableConstraintPrimaryKey(TypedDict)
```

`TableConstraint` member `primaryKey`.

`PRIMARY KEY (...)` columns.

**Attributes**

- `kind`: Always `"primaryKey"`.
- `value`: `PRIMARY KEY (...)` columns.

### TableConstraintUnique

```python
class TableConstraintUnique(TypedDict)
```

`TableConstraint` member `unique`.

`UNIQUE (...)` columns.

**Attributes**

- `kind`: Always `"unique"`.
- `value`: `UNIQUE (...)` columns.

### TableConstraintCheck

```python
class TableConstraintCheck(TypedDict)
```

`TableConstraint` member `check`.

`CHECK (...)` expression text.

**Attributes**

- `kind`: Always `"check"`.
- `value`: `CHECK (...)` expression text.

### TableConstraintForeignKey

```python
class TableConstraintForeignKey(TypedDict)
```

`TableConstraint` member `foreignKey`.

`FOREIGN KEY (...) REFERENCES ...`.

**Attributes**

- `kind`: Always `"foreignKey"`.
- `value`: `FOREIGN KEY (...) REFERENCES ...`.

### CreateTableSchema

```python
class CreateTableSchema(TypedDict)
```

Parsed `CREATE TABLE` (canonical SQL v1). Per-column lists align with `columns`.

**Attributes**

- `table`: Table name.
- `columns`: Columns with resolved types, in order.
- `identityColumn`: `INTEGER PRIMARY KEY AUTOINCREMENT` column; empty when none.
- `columnNotNull`: Per-column NOT NULL flags.
- `columnUnique`: Per-column UNIQUE flags.
- `columnPrimaryKey`: Per-column PRIMARY KEY flags.
- `columnDefaults`: Per-column DEFAULT expression text; empty when none.
- `columnChecks`: Per-column CHECK expression text; empty when none.
- `columnReferences`: Per-column REFERENCES targets.
- `tableConstraints`: Table-level constraints, in order.

### SchemaCreate

```python
class SchemaCreate(TypedDict)
```

`CREATE TABLE` action with its durable fingerprint.

**Attributes**

- `schema`: Parsed table schema.
- `fingerprint`: Structured schema fingerprint (hex SHA-256).
- `noop`: True when the catalog already holds this exact fingerprint.

### SchemaActionNone

```python
class SchemaActionNone(TypedDict)
```

`SchemaAction` member `none`.

Not DDL.

**Attributes**

- `kind`: Always `"none"`.

### SchemaActionCreate

```python
class SchemaActionCreate(TypedDict)
```

`SchemaAction` member `create`.

`CREATE TABLE`.

**Attributes**

- `kind`: Always `"create"`.
- `value`: `CREATE TABLE`.

### SchemaActionDrop

```python
class SchemaActionDrop(TypedDict)
```

`SchemaAction` member `drop`.

`DROP TABLE` of the named table.

**Attributes**

- `kind`: Always `"drop"`.
- `value`: `DROP TABLE` of the named table.

### ResolvedStatement

```python
class ResolvedStatement(TypedDict)
```

Host-produced typed proof bound to one exact canonical statement.

**Attributes**

- `statementHash`: SHA-256 hex of the exact canonical SQL this proof claims.
- `outputColumns`: SELECT / RETURNING / VALUES output columns in order.
- `physicalAccesses`: Physical tables/columns referenced (authorization).
- `assignments`: Mutation destination assignments.
- `textCollateSites`: TEXT expression spans needing bytewise collation.
- `integerArithSites`: INTEGER overflow sites (`+` `-` `*` `abs`).
- `functions`: Function names invoked (folded), for authorization.
- `schemaAction`: DDL action + fingerprint.

### AdapterReceipt

```python
class AdapterReceipt(TypedDict)
```

Replay receipt the host asks the adapter to persist with the batch.

**Attributes**

- `guestLen`: Guest statement count inside the receipt wrap (excluding host prune/select).
- `guestHash`: Guest `requestHash` compared on replay.

### AdapterStatement

```python
class AdapterStatement(TypedDict)
```

One host-lowered statement with its proof.

**Attributes**

- `sql`: Canonical SQL text.
- `parameters`: Positional bind values.
- `kind`: Statement classification.
- `maxRows`: Row cap for queries; `0` means adapter default.
- `resultSelection`: Which outcome parts to return.
- `proof`: Typed proof bound to `sql`.

### AdapterExecuteRequest

```python
class AdapterExecuteRequest(TypedDict)
```

Host -> adapter batch: proven statements plus isolation and receipt.

**Attributes**

- `operationId`: Host-chosen idempotency key.
- `requestHash`: SHA-256 hex of the idempotency-relevant request.
- `statements`: Statements in execution order.
- `deadlineUnixMs`: Deadline hint; `0` means none.
- `isolation`: Transaction isolation the adapter must realize.
- `receipt`: Replay receipt to persist; zero/empty when not required.

### StatementResult

```python
class StatementResult(TypedDict)
```

Outcome of one statement.

**Attributes**

- `rows`: Result rows (empty unless `rows` selected).
- `columns`: Result column descriptors.
- `rowsAffected`: Rows changed by a mutation.

### DbTiming

```python
class DbTiming(TypedDict)
```

Engine timing on `ExecuteReply`.

**Attributes**

- `attemptElapsedUs`: Monotonic duration of this handler attempt (microseconds).
- `dbExecutionUs`: Engine-reported SQL/transaction time when available (`0` = omitted).
- `dbTimingSource`: How `dbExecutionUs` was measured.

### ExecuteReply

```python
class ExecuteReply(TypedDict)
```

Success payload of `execute`.

**Attributes**

- `operationId`: Echo of the request `operationId`.
- `statements`: Per-statement results in request order.
- `timing`: Engine timing.

### ExecuteResultReplyOk

```python
class ExecuteResultReplyOk(TypedDict)
```

`ExecuteResultReply` member `ok`.

Success: statement results.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: statement results.

### ExecuteResultReplyErr

```python
class ExecuteResultReplyErr(TypedDict)
```

`ExecuteResultReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### DbCapabilities

```python
class DbCapabilities(TypedDict)
```

Semantic SQL-contract advertisement. Diagnostic engine identity is not
part of the capability plane — see `DbBootstrap`.

**Attributes**

- `sqlContractVersion`: Bookclerk SQL contract version.
- `atomicBatch`: Guest can run a bounded statement list as one SQL transaction.
- `returning`: Guest SQL supports `RETURNING`.
- `affectedRows`: Guest reports `rowsAffected`.
- `schemaMigrations`: Guest versions schema with a `bookclerk_schema_migrations` table.
- `cancellation`: Guest honors RPC/session cancellation.
- `timing`: Guest can fill `DbTiming.dbExecutionUs`. Not a host connect minimum.
- `maxBinds`: Maximum bound parameters per statement.
- `maxStatements`: Maximum statements in one atomic batch.
- `maxResultRows`: Maximum rows a query statement may return.
- `maxPayloadBytes`: Maximum UTF-8 bytes of SQL plus binds per statement.
- `maxResultBytes`: Maximum encoded bytes of one statement's result rows.
- `maxCellBytes`: Maximum UTF-8 / blob bytes of one result cell.
- `maxRequestBytes`: Maximum encoded bytes of one `ExecuteRequest`.
- `maxAtomicResultBytes`: Maximum encoded bytes of one `ExecuteReply`.
- `pluginDatabases`: Append-only. Adapter can open additional isolated sessions for plugin-owned database bindings (per-binding file / schema / database).
- `maxFunctionArgs`: Maximum arguments in one physical function call after adapter hiding (nested min / max / coalesce). Portable json_object is already ≤ 32 arguments (16 pairs), matching D1. `0` is unspecified.
- `maxSchemaColumns`: Maximum columns in one CREATE TABLE / result row. `0` is unspecified.
- `maxPatternBytes`: Maximum UTF-8 bytes of a BookclerkSQL LIKE pattern value (literals and TEXT binds). Adapters that expand LIKE into GLOB must advertise a conservative value that still fits the physical pattern cap. `0` is unspecified.
- `maxLoweredStatementBytes`: Maximum UTF-8 bytes of sqlite-family lowered SQL the adapter can realize for one statement (INTEGER overflow wraps, LIKE→GLOB, NULLIF, INSERT OR IGNORE, query LIMIT wrap, bytes-placeholder expansion). `0` is unspecified: the host does not enforce a lowered-size ceiling. First- party D1 advertises 100000. Hosts compare a standardized Bookclerk lowering upper bound against this number and must not branch on engine identity.
- `consistentBackupRead`: Adapter can expose one stable logical database state while the host reads schema, rows, and identity.
- `atomicUnitRestore`: Adapter can destructively replace one logical database unit so an ordinary restore failure does not leave that unit partially replaced.

### DbBootstrapReplyOk

```python
class DbBootstrapReplyOk(TypedDict)
```

`DbBootstrapReply` member `ok`.

Success: bootstrap metadata.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: bootstrap metadata.

### DbBootstrapReplyErr

```python
class DbBootstrapReplyErr(TypedDict)
```

`DbBootstrapReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### DbBootstrap

```python
class DbBootstrap(TypedDict)
```

Bootstrap-only diagnostic metadata (not a capability).

**Attributes**

- `engine`: Diagnostic physical engine name. Hosts must not admit or generate SQL from this value. Any string is valid.

### DbCapabilitiesReplyOk

```python
class DbCapabilitiesReplyOk(TypedDict)
```

`DbCapabilitiesReply` member `ok`.

Success: capability advertisement.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: capability advertisement.

### DbCapabilitiesReplyErr

```python
class DbCapabilitiesReplyErr(TypedDict)
```

`DbCapabilitiesReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### Database

```python
class Database(Protocol)
```

`databaseAdapter` entrypoint (`Entrypoints.databaseAdapter`).

#### Database.open_session

```python
async def open_session(self) -> AdapterSessionReply
```

Open one adapter session (capability negotiation + typed execute).

**Returns**

`AdapterSessionReply`

#### Database.drop_unit

```python
async def drop_unit(self, unit_ref: str) -> EmptyReply
```

Physically drop one provisioned binding unit (`unitRef` is adapter-owned).

**Args**

- `unit_ref`: Adapter-native unit (sqlite path, postgres database, D1 name).

**Returns**

`EmptyReply`

### IdentityHighWater

```python
class IdentityHighWater(TypedDict)
```

Adapter-private identity high-water (sqlite_sequence / bookclerk_identity).
Column names live in the canonical backup schema, not this catalog.

**Attributes**

- `table`: Table whose identity column the mark belongs to.
- `last`: Highest generated or stored value that must not be reused.

### IdentityExportReplyOk

```python
class IdentityExportReplyOk(TypedDict)
```

`IdentityExportReply` member `ok`.

Success: identity high-water rows.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: identity high-water rows.

### IdentityExportReplyErr

```python
class IdentityExportReplyErr(TypedDict)
```

`IdentityExportReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### UserRelationsReplyOk

```python
class UserRelationsReplyOk(TypedDict)
```

`UserRelationsReply` member `ok`.

Success: user relation names.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: user relation names.

### UserRelationsReplyErr

```python
class UserRelationsReplyErr(TypedDict)
```

`UserRelationsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### AdapterDatabaseSession

```python
class AdapterDatabaseSession(Protocol)
```

Host ↔ database adapter plugin. Capability negotiation + typed execute only.

#### AdapterDatabaseSession.capabilities

```python
async def capabilities(self) -> DbCapabilitiesReply
```

Semantic SQL-contract advertisement for this session.

**Returns**

`DbCapabilitiesReply`

#### AdapterDatabaseSession.execute

```python
async def execute(self, request: AdapterExecuteRequest) -> ExecuteResultReply
```

Canonical SQL + required structured proofs (not JSON).

**Args**

- `request`: Proven statements plus isolation and receipt.

**Returns**

`ExecuteResultReply`

#### AdapterDatabaseSession.close

```python
async def close(self) -> EmptyReply
```

Release the session; further calls fail.

**Returns**

`EmptyReply`

#### AdapterDatabaseSession.bootstrap

```python
async def bootstrap(self) -> DbBootstrapReply
```

Bootstrap-only SeaORM proxy metadata (not part of DbCapabilities).

**Returns**

`DbBootstrapReply`

#### AdapterDatabaseSession.export_identity

```python
async def export_identity(self) -> IdentityExportReply
```

Snapshot/identity/restore primitives (not a SQL dialect API).

**Returns**

`IdentityExportReply`

#### AdapterDatabaseSession.import_identity

```python
async def import_identity(self, rows: list[IdentityHighWater]) -> EmptyReply
```

Restore identity high-water marks captured by `exportIdentity`.

**Args**

- `rows`: High-water rows to apply.

**Returns**

`EmptyReply`

#### AdapterDatabaseSession.list_user_relations

```python
async def list_user_relations(self) -> UserRelationsReply
```

Names of user relations present in the logical database unit.

**Returns**

`UserRelationsReply`

#### AdapterDatabaseSession.prepare_unit_restore

```python
async def prepare_unit_restore(self) -> EmptyReply
```

Enter restore mode for one logical unit (`atomicUnitRestore`).

**Returns**

`EmptyReply`

#### AdapterDatabaseSession.drop_user_relations

```python
async def drop_user_relations(self, names: list[str]) -> EmptyReply
```

Drop the named user relations during restore.

**Args**

- `names`: Relation names from `listUserRelations`.

**Returns**

`EmptyReply`

#### AdapterDatabaseSession.assert_restore_constraints

```python
async def assert_restore_constraints(self) -> EmptyReply
```

Verify constraints hold after restore rows were written.

**Returns**

`EmptyReply`

### GuestDatabase

```python
class GuestDatabase(Protocol)
```

Host-granted SQL for job plugin authors. SDK `DatabaseBinding` mirrors the
Cloudflare Workers D1 surface (`prepare`/`bind`/`run`/`first`/`all`/`raw`,
`batch`, `exec`) over this typed `execute` transport; wire types stay Cap'n
`ExecuteRequest`/`ExecuteReply`.

#### GuestDatabase.execute

```python
async def execute(self, request: ExecuteRequest) -> ExecuteResultReply
```

Run one typed statement batch.

**Args**

- `request`: Statements, binds, and deadline.

**Returns**

`ExecuteResultReply`

#### GuestDatabase.close

```python
async def close(self) -> EmptyReply
```

Release the session; further calls fail.

**Returns**

`EmptyReply`

### PluginMigrationOpSchema

```python
class PluginMigrationOpSchema(TypedDict)
```

`PluginMigrationOp` member `schema`.

Admitted DDL statement text.

**Attributes**

- `kind`: Always `"schema"`.
- `value`: Admitted DDL statement text.

### PluginMigrationOpData

```python
class PluginMigrationOpData(TypedDict)
```

`PluginMigrationOp` member `data`.

Admitted DML statement text.

**Attributes**

- `kind`: Always `"data"`.
- `value`: Admitted DML statement text.

### PluginMigration

```python
class PluginMigration(TypedDict)
```

One plugin-owned migration application. `id` is an opaque plugin-chosen
stable identity (name, UUID, timestamp-like string, or digits-as-text).
Bookclerk assigns no order, version, or predecessor meaning to `id`.
Registration order is the forward sequence.

**Attributes**

- `id`: Opaque plugin-chosen stable identity.
- `operations`: Operations in forward order; at most `maxPluginMigrationOps`.

### PluginMigrationsOk

```python
class PluginMigrationsOk(TypedDict)
```

Success payload of `PluginWorker.databaseMigrations`.

**Attributes**

- `migrations`: At most `maxListPage` entries; aggregate id+SQL bytes at most `maxPluginMigrationRegistrationBytes`; total operations at most `maxPluginMigrationTotalOps`.

### PluginMigrationsReplyOk

```python
class PluginMigrationsReplyOk(TypedDict)
```

`PluginMigrationsReply` member `ok`.

Success: ordered migration sequence.

**Attributes**

- `kind`: Always `"ok"`.
- `value`: Success: ordered migration sequence.

### PluginMigrationsReplyErr

```python
class PluginMigrationsReplyErr(TypedDict)
```

`PluginMigrationsReply` member `err`.

Typed failure; `code` is a `PluginErrorCode` wire string.

**Attributes**

- `kind`: Always `"err"`.
- `value`: Typed failure; `code` is a `PluginErrorCode` wire string.

### PluginWorker

```python
class PluginWorker(Protocol)
```

Plugin bootstrap capability: the guest's root object (Workers `default`
export analogue). `describe()` first; `open()` once per invocation.

#### PluginWorker.describe

```python
async def describe(self) -> DescribeReply
```

Identity, ABI version, negotiated features, and typed capabilities.

**Returns**

`DescribeReply`

#### PluginWorker.open

```python
async def open(self, invocation: Invocation, bindings: Bindings) -> EntrypointsReply
```

Open the exported entrypoints for one invocation with host-granted
bindings. Entrypoints the manifest does not export are null.

**Args**

- `invocation`: Host-issued invocation identity.
- `bindings`: Granted `env` bindings.

**Returns**

`EntrypointsReply`

#### PluginWorker.shutdown

```python
async def shutdown(self) -> EmptyReply
```

Flush and release resources before the process exits.

**Returns**

`EmptyReply`

#### PluginWorker.database_migrations

```python
async def database_migrations(self, binding: str) -> PluginMigrationsReply
```

Complete ordered plugin-owned migration sequence for one named binding.
The host calls this before `open` while provisioning `[[databases]]`
(a binding must be migrated before any `Bindings.databases` session is
handed out). Empty list means the binding has no plugin-owned migrations.
Bounded by `maxListPage` / `maxPluginMigrationOps` /
`maxPluginMigrationTotalOps` / `maxScalarBytes` /
`maxPluginMigrationRegistrationBytes`.

**Args**

- `binding`: Binding name from `plugin.toml`.

**Returns**

`PluginMigrationsReply`

## Constants

### PluginErrorCode

```python
PluginErrorCode = Literal['invalid_params', 'unauthorized', 'forbidden', 'not_found', 'unavaila...
```

Stable `PluginError.code` strings. Unknown future codes are forwarded
as-is; SDKs surface them as a local `unknown` while keeping the raw wire
code.

### Entrypoint

```python
Entrypoint = Literal['storefront', 'storage', 'databaseAdapter', 'remoteLibrary', 'cli', '...
```

Named entrypoint a plugin exports (Cloudflare Workers named-entrypoint
analogue). Each value is a capability the host calls over RPC; triggers on
the default entrypoint (`event`, `job`) are declared separately.

### PortalAuthMode

```python
PortalAuthMode = Literal['unspecified', 'password', 'oauth']
```

Portal Accounts connect mode for storefronts.

### CliArgKind

```python
CliArgKind = Literal['string', 'bool', 'int', 'path']
```

Value kind for a `CliArgSpec`.

### CatalogSort

```python
CatalogSort = Literal['relevance', 'popularity', 'rating', 'title', 'author']
```

Catalog search ordering.

### CatalogField

```python
CatalogField = Literal['any', 'author', 'narrator', 'series', 'genre']
```

Catalog search facet restricting which field the query matches.

### Abridgement

```python
Abridgement = Literal['unknown', 'unabridged', 'abridged']
```

Whether an edition is abridged.

### DbType

```python
DbType = Literal['unspecified', 'bool', 'int64', 'float64', 'text', 'bytes']
```

Universal database cell/parameter domain. Engine-native arrays, enums,
unsigned integers, and JSON text sentinels are not baseline ABI values.

### DbStatementKind

```python
DbStatementKind = Literal['execute', 'select', 'returning']
```

How the host classifies a statement for result handling.

### DbResultSelection

```python
DbResultSelection = Literal['discard', 'affectedRows', 'rows']
```

Which part of a statement's outcome the caller wants back.

### IsolationReq

```python
IsolationReq = Literal['atomicBatch', 'nestedSavepoint', 'consistentSnapshot']
```

Host → adapter execute. GuestDatabase stays ExecuteRequest-only.

### ResolvedSqlType

```python
ResolvedSqlType = Literal['integer', 'real', 'text', 'blob', 'boolean', 'null']
```

Resolved BookclerkSQL column type after type checking.

### IntegerArithKind

```python
IntegerArithKind = Literal['add', 'sub', 'mul', 'abs']
```

INTEGER `+` `-` `*` `abs` site lowered to overflow -> NULL.

### OidcClientsReply

```python
OidcClientsReply = Union[OidcClientsReplyOk, OidcClientsReplyErr]
```

Result union of `Oidc.clients`.

### EntrypointsReply

```python
EntrypointsReply = Union[EntrypointsReplyOk, EntrypointsReplyErr]
```

Result union of `PluginWorker.open`.

### JobOutcome

```python
JobOutcome = Union[JobOutcomeCompleted, JobOutcomeRetryable, JobOutcomeRejected, JobOutcom...
```

Terminal or suspended result of `JobRunner.job`.

### EventResult

```python
EventResult = Union[EventResultAck, EventResultRetry, EventResultReject, EventResultDeadLet...
```

Per-event outcome inside an `EventConsumer.event` batch reply.

### EventBatchReply

```python
EventBatchReply = Union[EventBatchReplyOk, EventBatchReplyErr]
```

Result union of `EventConsumer.event`. `ok` has exactly one entry per
`EventBatch.events` entry, in order; a short list is a host-side error and
the missing tail is retried.

### HeadReply

```python
HeadReply = Union[HeadReplyOk, HeadReplyErr]
```

Result union of `Destination.head`.

### ListReply

```python
ListReply = Union[ListReplyOk, ListReplyErr]
```

Result union of `Destination.list`.

### GetReply

```python
GetReply = Union[GetReplyOk, GetReplyErr]
```

Result union of `Destination.get`.

### PutReply

```python
PutReply = Union[PutReplyOk, PutReplyErr]
```

Result union of `Destination.put` / `Destination.commit`.

### CopyReply

```python
CopyReply = Union[CopyReplyOk, CopyReplyErr]
```

Result union of `Destination.copy`.

### EmptyReply

```python
EmptyReply = Union[EmptyReplyOk, EmptyReplyErr]
```

Result union of `methods without a success payload`.

### PullReply

```python
PullReply = Union[PullReplyOk, PullReplyErr]
```

Result union of `ByteSource.pull`.

### OpenReply

```python
OpenReply = Union[OpenReplyOk, OpenReplyErr]
```

Result union of `Source.open`.

### DescribeReply

```python
DescribeReply = Union[DescribeReplyOk, DescribeReplyErr]
```

Result union of `PluginWorker.describe`.

### HandleReply

```python
HandleReply = Union[HandleReplyOk, HandleReplyErr]
```

Result union of `JobRunner.job`.

### HealthReply

```python
HealthReply = Union[HealthReplyOk, HealthReplyErr]
```

Result union of `health`.

### AdapterSessionReply

```python
AdapterSessionReply = Union[AdapterSessionReplyOk, AdapterSessionReplyErr]
```

Result union of `Database.openSession`.

### GuestDatabaseReply

```python
GuestDatabaseReply = Union[GuestDatabaseReplyOk, GuestDatabaseReplyErr]
```

Result union of `guest database opens`.

### CliSchemaReply

```python
CliSchemaReply = Union[CliSchemaReplyOk, CliSchemaReplyErr]
```

Result union of `PluginCli.describe`.

### CliInvokeReply

```python
CliInvokeReply = Union[CliInvokeReplyOk, CliInvokeReplyErr]
```

Result union of `PluginCli.invoke`.

### DiagnoseReply

```python
DiagnoseReply = Union[DiagnoseReplyOk, DiagnoseReplyErr]
```

Result union of `diagnose`.

### LoginReply

```python
LoginReply = Union[LoginReplyOk, LoginReplyErr]
```

Result union of `ContentSource.login` / `loginComplete`.

### LoginStartReply

```python
LoginStartReply = Union[LoginStartReplyOk, LoginStartReplyErr]
```

Result union of `ContentSource.loginStart`.

### ScanReply

```python
ScanReply = Union[ScanReplyOk, ScanReplyErr]
```

Result union of `ContentSource.scan`.

### FetchTitleReply

```python
FetchTitleReply = Union[FetchTitleReplyOk, FetchTitleReplyErr]
```

Result union of `ContentSource.fetchTitle`.

### SourceAccountsReply

```python
SourceAccountsReply = Union[SourceAccountsReplyOk, SourceAccountsReplyErr]
```

Result union of `ContentSource.listAccounts`.

### CatalogHitsReply

```python
CatalogHitsReply = Union[CatalogHitsReplyOk, CatalogHitsReplyErr]
```

Result union of `searchCatalog` / `expandCandidates` / `listDeals`.

### CatalogDetailReply

```python
CatalogDetailReply = Union[CatalogDetailReplyOk, CatalogDetailReplyErr]
```

Result union of `ContentSource.catalogDetail`.

### PurchaseHintReply

```python
PurchaseHintReply = Union[PurchaseHintReplyOk, PurchaseHintReplyErr]
```

Result union of `ContentSource.purchaseHint`.

### ExternalUserReply

```python
ExternalUserReply = Union[ExternalUserReplyOk, ExternalUserReplyErr]
```

Result union of `Oidc.authenticateUser`.

### EventPollReply

```python
EventPollReply = Union[EventPollReplyOk, EventPollReplyErr]
```

Result union of `RemoteLibrary.pollEvents`.

### SyncListeningReply

```python
SyncListeningReply = Union[SyncListeningReplyOk, SyncListeningReplyErr]
```

Result union of `RemoteLibrary.syncListening`.

### PublishReply

```python
PublishReply = Union[PublishReplyOk, PublishReplyErr]
```

Result union of `EventPublisher.publish`.

### QueueResult

```python
QueueResult = Union[QueueResultAck, QueueResultRetry]
```

Per-message outcome inside a `QueueConsumer.queue` batch reply.

### QueueBatchReply

```python
QueueBatchReply = Union[QueueBatchReplyOk, QueueBatchReplyErr]
```

Result union of `QueueConsumer.queue`.

### QueueSendReply

```python
QueueSendReply = Union[QueueSendReplyOk, QueueSendReplyErr]
```

Result union of `QueueProducer.send` / `sendBatch`.

### KvGetReply

```python
KvGetReply = Union[KvGetReplyOk, KvGetReplyErr]
```

Result union of `KvNamespace.get`.

### KvPutReply

```python
KvPutReply = Union[KvPutReplyOk, KvPutReplyErr]
```

Result union of `KvNamespace.put`.

### KvDeleteReply

```python
KvDeleteReply = Union[KvDeleteReplyOk, KvDeleteReplyErr]
```

Result union of `KvNamespace.delete`.

### KvListReply

```python
KvListReply = Union[KvListReplyOk, KvListReplyErr]
```

Result union of `KvNamespace.list`.

### DbValue

```python
DbValue = Union[DbValueNull, DbValueBoolean, DbValueInt64, DbValueFloat64, DbValueText,...
```

One typed SQL cell or bind parameter.

### OptionalColumnReference

```python
OptionalColumnReference = Union[OptionalColumnReferenceNone, OptionalColumnReferenceSome]
```

Per-column `REFERENCES` slot in `CreateTableSchema`.

### TableConstraint

```python
TableConstraint = Union[TableConstraintPrimaryKey, TableConstraintUnique, TableConstraintCheck,...
```

One table-level constraint.

### SchemaAction

```python
SchemaAction = Union[SchemaActionNone, SchemaActionCreate, SchemaActionDrop]
```

CREATE/DROP action recorded on a proof.

### ExecuteResultReply

```python
ExecuteResultReply = Union[ExecuteResultReplyOk, ExecuteResultReplyErr]
```

Result union of `execute`.

### DbBootstrapReply

```python
DbBootstrapReply = Union[DbBootstrapReplyOk, DbBootstrapReplyErr]
```

Result union of `AdapterDatabaseSession.bootstrap`.

### DbCapabilitiesReply

```python
DbCapabilitiesReply = Union[DbCapabilitiesReplyOk, DbCapabilitiesReplyErr]
```

Result union of `AdapterDatabaseSession.capabilities`.

### IdentityExportReply

```python
IdentityExportReply = Union[IdentityExportReplyOk, IdentityExportReplyErr]
```

Result union of `AdapterDatabaseSession.exportIdentity`.

### UserRelationsReply

```python
UserRelationsReply = Union[UserRelationsReplyOk, UserRelationsReplyErr]
```

Result union of `AdapterDatabaseSession.listUserRelations`.

### PluginMigrationOp

```python
PluginMigrationOp = Union[PluginMigrationOpSchema, PluginMigrationOpData]
```

One already-separated BookclerkSQL operation in a plugin-owned migration.
`schema` is admitted DDL; `data` is admitted DML. There is no native SQL
escape hatch.

### PluginMigrationsReply

```python
PluginMigrationsReply = Union[PluginMigrationsReplyOk, PluginMigrationsReplyErr]
```

Result union of `PluginWorker.databaseMigrations`.
