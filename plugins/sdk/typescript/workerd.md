# workerd

## Classes

### InvokeError

**`Internal`**

Transport-level `/invoke` failure (never an author failure): unknown
interface/method, malformed message, bad capability table, or a missing
binding.

#### Extends

- [`PluginError`](README.md#pluginerror)

#### Constructors

##### Constructor

```ts
new InvokeError(
   status, 
   code, 
   message
): InvokeError;
```

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `status` | `number` |
| `code` | `string` |
| `message` | `string` |

###### Returns

[`InvokeError`](#invokeerror)

###### Overrides

[`PluginError`](README.md#pluginerror).[`constructor`](README.md#constructor-1)

#### Properties

##### code

```ts
readonly code: string;
```

Known `PluginErrorCode` wire string, or `unknown`.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`code`](README.md#code)

##### details

```ts
readonly details: Readonly<Record<string, unknown>> | undefined;
```

Structured detail object from the wire (`details` JSON), when the error carries one.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`details`](README.md#details)

##### status

```ts
readonly status: number;
```

HTTP status the bridge answers with.

##### wireCode

```ts
readonly wireCode: string;
```

Raw wire code, including codes this SDK does not know.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`wireCode`](README.md#wirecode)

#### Accessors

##### detailsText

###### Get Signature

```ts
get detailsText(): string;
```

`details` as the JSON text the wire carries.

###### Returns

`string`

JSON object text, or an empty string when the error has none.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`detailsText`](README.md#detailstext)

##### limit

###### Get Signature

```ts
get limit(): LimitExceeded | undefined;
```

The exceeded cap when this is a `limit_exceeded` error.

###### Returns

[`LimitExceeded`](README.md#limitexceeded-1) \| `undefined`

The cap from `details` first, then from the canonical message;
`undefined` for every other code.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`limit`](README.md#limit)

#### Methods

##### fromWire()

```ts
static fromWire(
   code, 
   message, 
   details?
): PluginError;
```

Construct a [PluginError](README.md#pluginerror) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](README.md#code) while
[PluginError.wireCode](README.md#wirecode) keeps the raw value.

###### Parameters

| Parameter | Type | Default value | Description |
| ------ | ------ | ------ | ------ |
| `code` | `string` | `undefined` | Wire error code (known or unknown). |
| `message` | `string` | `undefined` | Operator-facing error text. |
| `details?` | `string` | `""` | `details` JSON text from the wire (empty when none). |

###### Returns

[`PluginError`](README.md#pluginerror)

Typed plugin error.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`fromWire`](README.md#fromwire)

##### limitExceeded()

```ts
static limitExceeded(limit): PluginError;
```

Builds a `limit_exceeded` error whose message and `details` both carry
the exceeded cap.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `limit` | [`LimitExceeded`](README.md#limitexceeded-1) | Exceeded cap. |

###### Returns

[`PluginError`](README.md#pluginerror)

Typed plugin error.

###### Inherited from

[`PluginError`](README.md#pluginerror).[`limitExceeded`](README.md#limitexceeded)

## Interfaces

### CapDescriptor

**`Internal`**

One `X-Bookclerk-Caps` entry: capability index `i` of the message is
described by entry `i`.

#### Properties

##### binding?

```ts
binding?: string;
```

Binding name for `guestDatabase`.

##### id?

```ts
id?: string;
```

Isolate object id for `adapterSession`.

##### kind

```ts
kind: string;
```

`source` / `destination` / `progress` / `cancellation` / … / `adapterSession`.

##### token?

```ts
token?: string;
```

Grant token for host-served kinds.

***

### InvokeFailure

**`Internal`**

Transport-level `/invoke` failure as a plain value (Workers RPC drops own
properties of thrown errors, so the adapter returns this instead of
throwing across the service binding).

#### Properties

##### error

```ts
error: object;
```

JSON error body.

###### code

```ts
code: string;
```

###### message

```ts
message: string;
```

##### status

```ts
status: number;
```

HTTP status the bridge answers with (`400` / `404` / `500`).

***

### InvokeHost

**`Internal`**

Adapter hooks the dispatcher calls back into.

#### Methods

##### author()

```ts
author(): AuthorStub | undefined;
```

Default author entrypoint (`PLUGIN`), or `undefined` when unbound.

###### Returns

[`AuthorStub`](plugin.md#authorstub) \| `undefined`

##### bindContext()

```ts
bindContext(ctx): GrantedContext;
```

Bridge context → author-facing granted context.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | [`BridgeContext`](plugin.md#bridgecontext) \| `undefined` |

###### Returns

[`GrantedContext`](plugin.md#grantedcontext)

##### importCap()

```ts
importCap(descriptor, stop): unknown;
```

Resolve one request capability descriptor; `undefined` for an unknown
kind. `stop` aborts when the call returns.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `descriptor` | [`CapDescriptor`](#capdescriptor) |
| `stop` | `AbortSignal` |

###### Returns

`unknown`

##### named()

```ts
named(name): NamedStub | undefined;
```

Named author entrypoint, or `undefined` when the isolate does not export it.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `name` | [`EntrypointName`](plugin.md#entrypointname) |

###### Returns

[`NamedStub`](plugin.md#namedstub) \| `undefined`

***

### InvokeReply

**`Internal`**

Successful `/invoke` reply: `$Results` bytes plus the descriptors of
capabilities the reply exported.

#### Properties

##### body

```ts
body: Uint8Array;
```

Unpacked single-segment Cap'n message (`<Interface>.<method>$Results`).

##### caps

```ts
caps: CapDescriptor[];
```

Reply capability table (`X-Bookclerk-Caps` on the response).

## Type Aliases

### InvokeOutcome

```ts
type InvokeOutcome = InvokeReply | InvokeFailure;
```

**`Internal`**

What the adapter's `invoke` resolves to.

## Functions

### dispatchInvoke()

```ts
function dispatchInvoke(
   iface, 
   method, 
   context, 
   caps, 
   target, 
   body, 
   host
): Promise<InvokeReply>;
```

**`Internal`**

Dispatch one `/invoke` call.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `iface` | `string` | Cap'n interface name (`X-Bookclerk-Interface`). |
| `method` | `string` | Method name (`X-Bookclerk-Method`). |
| `context` | [`BridgeContext`](plugin.md#bridgecontext) \| `undefined` | Decoded bridge context, or `undefined` for `PluginWorker.*`. |
| `caps` | [`CapDescriptor`](#capdescriptor)[] | Request capability descriptors (`X-Bookclerk-Caps`). |
| `target` | `string` \| `null` | Isolate object id (`X-Bookclerk-Target`), or `null`. |
| `body` | `Uint8Array` | `$Params` message bytes. |
| `host` | [`InvokeHost`](#invokehost) | Adapter hooks. |

#### Returns

`Promise`\<[`InvokeReply`](#invokereply)\>

`$Results` bytes plus reply capability descriptors.

#### Throws

On transport-level failures (`400` / `404` / `500`).

## References

### Abridgement

Re-exports [Abridgement](abi.md#abridgement)

***

### AdapterBatchOptions

Re-exports [AdapterBatchOptions](README.md#adapterbatchoptions)

***

### AdapterBatchRequest

Re-exports [AdapterBatchRequest](README.md#adapterbatchrequest)

***

### AdapterBatchStatement

Re-exports [AdapterBatchStatement](README.md#adapterbatchstatement)

***

### AdapterBatchStreamedOptions

Re-exports [AdapterBatchStreamedOptions](README.md#adapterbatchstreamedoptions)

***

### AdapterDatabaseSession

Re-exports [AdapterDatabaseSession](plugin.md#adapterdatabasesession)

***

### AdapterEnv

Re-exports [AdapterEnv](plugin.md#adapterenv)

***

### AdapterTransaction

Re-exports [AdapterTransaction](README.md#adaptertransaction)

***

### allowedEntrypointFamilies

Re-exports [allowedEntrypointFamilies](plugin.md#allowedentrypointfamilies)

***

### AtomicTransport

Re-exports [AtomicTransport](db-execute.md#atomictransport)

***

### BookclerkEntrypoint

Re-exports [BookclerkEntrypoint](plugin.md#bookclerkentrypoint)

***

### BookclerkEnv

Re-exports [BookclerkEnv](env.md#bookclerkenv)

***

### canonicalExecuteRequestHash

Re-exports [canonicalExecuteRequestHash](db-execute.md#canonicalexecuterequesthash)

***

### Checkpoint

Re-exports [Checkpoint](plugin.md#checkpoint-4)

***

### chunkIndexes

Re-exports [chunkIndexes](db-execute.md#chunkindexes)

***

### cliArgs

Re-exports [cliArgs](plugin.md#cliargs)

***

### CliEntrypoint

Re-exports [CliEntrypoint](plugin.md#clientrypoint)

***

### CliInvokeParams

Re-exports [CliInvokeParams](generated.md#cliinvokeparams)

***

### CliInvokeResult

Re-exports [CliInvokeResult](generated.md#cliinvokeresult)

***

### CliSchema

Re-exports [CliSchema](generated.md#clischema)

***

### collectExecuteStream

Re-exports [collectExecuteStream](README.md#collectexecutestream)

***

### CopyResult

Re-exports [CopyResult](plugin.md#copyresult)

***

### CoverEntrypoint

Re-exports [CoverEntrypoint](plugin.md#coverentrypoint)

***

### CoverFetchParams

Re-exports [CoverFetchParams](generated.md#coverfetchparams)

***

### CoverImage

Re-exports [CoverImage](generated.md#coverimage)

***

### CoverImages

Re-exports [CoverImages](generated.md#coverimages)

***

### createDatabaseBinding

Re-exports [createDatabaseBinding](db-execute.md#createdatabasebinding)

***

### Credit

Re-exports [Credit](generated.md#credit)

***

### CreditLevel

Re-exports [CreditLevel](abi.md#creditlevel)

***

### CreditRole

Re-exports [CreditRole](abi.md#creditrole)

***

### D1ExecResult

Re-exports [D1ExecResult](db-execute.md#d1execresult)

***

### D1Meta

Re-exports [D1Meta](db-execute.md#d1meta)

***

### D1Result

Re-exports [D1Result](db-execute.md#d1result)

***

### DatabaseAdapterEntrypoint

Re-exports [DatabaseAdapterEntrypoint](plugin.md#databaseadapterentrypoint)

***

### DatabaseBinding

Re-exports [DatabaseBinding](db-execute.md#databasebinding)

***

### DatabaseBindingOptions

Re-exports [DatabaseBindingOptions](db-execute.md#databasebindingoptions)

***

### dataMigrationOp

Re-exports [dataMigrationOp](plugin.md#datamigrationop)

***

### DbColumn

Re-exports [DbColumn](db-execute.md#dbcolumn)

***

### DbResultSelection

Re-exports [DbResultSelection](abi.md#dbresultselection)

***

### DbRow

Re-exports [DbRow](db-execute.md#dbrow)

***

### DbStatementKind

Re-exports [DbStatementKind](abi.md#dbstatementkind)

***

### DbTiming

Re-exports [DbTiming](db-execute.md#dbtiming)

***

### DbType

Re-exports [DbType](abi.md#dbtype)

***

### DbValue

Re-exports [DbValue](db-value.md#dbvalue)

***

### decodeDbValue

Re-exports [decodeDbValue](db-value.md#decodedbvalue)

***

### decodeExecuteRequest

Re-exports [decodeExecuteRequest](db-execute.md#decodeexecuterequest)

***

### decodeExecuteResultReply

Re-exports [decodeExecuteResultReply](db-execute.md#decodeexecuteresultreply)

***

### decodeExecuteStream

Re-exports [decodeExecuteStream](README.md#decodeexecutestream)

***

### decodeExtensibleConfig

Re-exports [decodeExtensibleConfig](plugin.md#decodeextensibleconfig)

***

### DeliveredCheckpoint

Re-exports [DeliveredCheckpoint](plugin.md#deliveredcheckpoint)

***

### Destination

Re-exports [Destination](plugin.md#destination)

***

### DomainEvent

Re-exports [DomainEvent](generated.md#domainevent)

***

### encodeDbValue

Re-exports [encodeDbValue](db-value.md#encodedbvalue)

***

### encodeExecuteRequest

Re-exports [encodeExecuteRequest](db-execute.md#encodeexecuterequest)

***

### encodeExecuteResultReply

Re-exports [encodeExecuteResultReply](db-execute.md#encodeexecuteresultreply)

***

### encodeExecuteStream

Re-exports [encodeExecuteStream](README.md#encodeexecutestream)

***

### ENTRYPOINT\_BINDINGS

Re-exports [ENTRYPOINT_BINDINGS](plugin.md#entrypoint_bindings)

***

### ENTRYPOINT\_CLASSES

Re-exports [ENTRYPOINT_CLASSES](plugin.md#entrypoint_classes)

***

### EntrypointName

Re-exports [EntrypointName](plugin.md#entrypointname)

***

### EventBatch

Re-exports [EventBatch](plugin.md#eventbatch)

***

### eventBatchResults

Re-exports [eventBatchResults](plugin.md#eventbatchresults)

***

### EventMessage

Re-exports [EventMessage](plugin.md#eventmessage)

***

### EventOutcome

Re-exports [EventOutcome](plugin.md#eventoutcome)

***

### EventPublisherBinding

Re-exports [EventPublisherBinding](env.md#eventpublisherbinding)

***

### EventSuspendOptions

Re-exports [EventSuspendOptions](plugin.md#eventsuspendoptions)

***

### executeAdapterBatch

Re-exports [executeAdapterBatch](README.md#executeadapterbatch)

***

### executeAdapterBatchStreamed

Re-exports [executeAdapterBatchStreamed](README.md#executeadapterbatchstreamed)

***

### ExecuteReply

Re-exports [ExecuteReply](db-execute.md#executereply)

***

### executeReplyToD1Results

Re-exports [executeReplyToD1Results](db-execute.md#executereplytod1results)

***

### ExecuteRequest

Re-exports [ExecuteRequest](db-execute.md#executerequest)

***

### ExecuteStreamDecoder

Re-exports [ExecuteStreamDecoder](README.md#executestreamdecoder)

***

### executeStreamLimits

Re-exports [executeStreamLimits](README.md#executestreamlimits-1)

***

### ExecuteStreamLimits

Re-exports [ExecuteStreamLimits](README.md#executestreamlimits)

***

### ExpressionKind

Re-exports [ExpressionKind](abi.md#expressionkind)

***

### ExtensibleConfig

Re-exports [ExtensibleConfig](generated.md#extensibleconfig)

***

### FEATURE\_SCALAR\_LIMITS

Re-exports [FEATURE_SCALAR_LIMITS](abi.md#feature_scalar_limits)

***

### FEATURE\_STORAGE\_COPY

Re-exports [FEATURE_STORAGE_COPY](abi.md#feature_storage_copy)

***

### FEATURE\_STREAMS

Re-exports [FEATURE_STREAMS](abi.md#feature_streams)

***

### fromBridgeJson

Re-exports [fromBridgeJson](plugin.md#frombridgejson)

***

### GrantedContext

Re-exports [GrantedContext](plugin.md#grantedcontext)

***

### GrantedDatabaseTransport

Re-exports [GrantedDatabaseTransport](plugin.md#granteddatabasetransport)

***

### GUEST\_READ\_ONLY\_REPLAY\_PAYLOAD

Re-exports [GUEST_READ_ONLY_REPLAY_PAYLOAD](README.md#guest_read_only_replay_payload)

***

### GUEST\_RECEIPT\_CLAIM\_LOST

Re-exports [GUEST_RECEIPT_CLAIM_LOST](README.md#guest_receipt_claim_lost)

***

### GUEST\_RECEIPT\_RESULT\_LOST

Re-exports [GUEST_RECEIPT_RESULT_LOST](README.md#guest_receipt_result_lost)

***

### GUEST\_RECEIPT\_STUB\_SUFFIX

Re-exports [GUEST_RECEIPT_STUB_SUFFIX](README.md#guest_receipt_stub_suffix)

***

### GUEST\_RECEIPT\_WRAP\_PREFIX

Re-exports [GUEST_RECEIPT_WRAP_PREFIX](README.md#guest_receipt_wrap_prefix)

***

### GUEST\_REPLAY\_PAYLOAD\_MAX\_BYTES

Re-exports [GUEST_REPLAY_PAYLOAD_MAX_BYTES](README.md#guest_replay_payload_max_bytes)

***

### guestBatchIsReadOnly

Re-exports [guestBatchIsReadOnly](README.md#guestbatchisreadonly)

***

### guestReceiptFinalizeStatements

Re-exports [guestReceiptFinalizeStatements](README.md#guestreceiptfinalizestatements)

***

### guestReplayPayload

Re-exports [guestReplayPayload](README.md#guestreplaypayload)

***

### HealthOk

Re-exports [HealthOk](generated.md#healthok)

***

### Identifier

Re-exports [Identifier](generated.md#identifier)

***

### IdentifierScheme

Re-exports [IdentifierScheme](abi.md#identifierscheme)

***

### Instant

Re-exports [Instant](plugin.md#instant)

***

### Invocation

Re-exports [Invocation](generated.md#invocation)

***

### invocationOf

Re-exports [invocationOf](plugin.md#invocationof)

***

### JobCheckpoint

Re-exports [JobCheckpoint](plugin.md#jobcheckpoint)

***

### JobCompletion

Re-exports [JobCompletion](plugin.md#jobcompletion)

***

### JobController

Re-exports [JobController](plugin.md#jobcontroller)

***

### JobInvocation

Re-exports [JobInvocation](generated.md#jobinvocation)

***

### jobOutcomeFor

Re-exports [jobOutcomeFor](plugin.md#joboutcomefor)

***

### JobOutcomeRecord

Re-exports [JobOutcomeRecord](plugin.md#joboutcomerecord)

***

### JobRetryOptions

Re-exports [JobRetryOptions](plugin.md#jobretryoptions)

***

### JobRunnerContext

Re-exports [JobRunnerContext](plugin.md#jobrunnercontext)

***

### JobSuspendOptions

Re-exports [JobSuspendOptions](plugin.md#jobsuspendoptions)

***

### JsonObject

Re-exports [JsonObject](env.md#jsonobject)

***

### jsonPayload

Re-exports [jsonPayload](plugin.md#jsonpayload)

***

### KvListKeyBinding

Re-exports [KvListKeyBinding](env.md#kvlistkeybinding)

***

### KvListOptionsBinding

Re-exports [KvListOptionsBinding](env.md#kvlistoptionsbinding)

***

### KvListResultBinding

Re-exports [KvListResultBinding](env.md#kvlistresultbinding)

***

### KvNamespaceBinding

Re-exports [KvNamespaceBinding](env.md#kvnamespacebinding)

***

### KvPutOptionsBinding

Re-exports [KvPutOptionsBinding](env.md#kvputoptionsbinding)

***

### KvValueType

Re-exports [KvValueType](env.md#kvvaluetype)

***

### KvValueWithMetadata

Re-exports [KvValueWithMetadata](env.md#kvvaluewithmetadata)

***

### ListOptions

Re-exports [ListOptions](plugin.md#listoptions)

***

### ListPage

Re-exports [ListPage](plugin.md#listpage)

***

### MAX\_CHECKPOINT\_BYTES

Re-exports [MAX_CHECKPOINT_BYTES](abi.md#max_checkpoint_bytes)

***

### MAX\_EVENT\_PAYLOAD\_BYTES

Re-exports [MAX_EVENT_PAYLOAD_BYTES](abi.md#max_event_payload_bytes)

***

### MAX\_LIST\_PAGE

Re-exports [MAX_LIST_PAGE](abi.md#max_list_page)

***

### MAX\_PLUGIN\_MIGRATION\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_OPS](abi.md#max_plugin_migration_ops)

***

### MAX\_PLUGIN\_MIGRATION\_REGISTRATION\_BYTES

Re-exports [MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES](abi.md#max_plugin_migration_registration_bytes)

***

### MAX\_PLUGIN\_MIGRATION\_TOTAL\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_TOTAL_OPS](abi.md#max_plugin_migration_total_ops)

***

### MAX\_SCALAR\_BYTES

Re-exports [MAX_SCALAR_BYTES](abi.md#max_scalar_bytes)

***

### MAX\_STREAM\_WINDOW\_BYTES

Re-exports [MAX_STREAM_WINDOW_BYTES](abi.md#max_stream_window_bytes)

***

### MAX\_WRITE\_KEY\_BYTES

Re-exports [MAX_WRITE_KEY_BYTES](db-execute.md#max_write_key_bytes)

***

### MediaKind

Re-exports [MediaKind](abi.md#mediakind)

***

### MetadataChapter

Re-exports [MetadataChapter](generated.md#metadatachapter)

***

### MetadataChapters

Re-exports [MetadataChapters](generated.md#metadatachapters)

***

### MetadataChaptersParams

Re-exports [MetadataChaptersParams](generated.md#metadatachaptersparams)

***

### MetadataEntrypoint

Re-exports [MetadataEntrypoint](plugin.md#metadataentrypoint)

***

### MetadataLookupParams

Re-exports [MetadataLookupParams](generated.md#metadatalookupparams)

***

### MetadataRecord

Re-exports [MetadataRecord](generated.md#metadatarecord)

***

### MetadataSearchParams

Re-exports [MetadataSearchParams](generated.md#metadatasearchparams)

***

### NamedDatabase

Re-exports [NamedDatabase](plugin.md#nameddatabase)

***

### NamedEntrypoint

Re-exports [NamedEntrypoint](plugin.md#namedentrypoint)

***

### NamedKv

Re-exports [NamedKv](plugin.md#namedkv)

***

### NamedQueue

Re-exports [NamedQueue](plugin.md#namedqueue)

***

### ObjectInfo

Re-exports [ObjectInfo](plugin.md#objectinfo)

***

### ObjectMetadata

Re-exports [ObjectMetadata](plugin.md#objectmetadata)

***

### OidcClientTemplate

Re-exports [OidcClientTemplate](generated.md#oidcclienttemplate)

***

### OidcEntrypoint

Re-exports [OidcEntrypoint](plugin.md#oidcentrypoint)

***

### parseDbValue

Re-exports [parseDbValue](db-value.md#parsedbvalue)

***

### PluginCapabilities

Re-exports [PluginCapabilities](generated.md#plugincapabilities)

***

### PluginDescribe

Re-exports [PluginDescribe](plugin.md#plugindescribe)

***

### PluginError

Re-exports [PluginError](README.md#pluginerror)

***

### PluginMigration

Re-exports [PluginMigration](generated.md#pluginmigration)

***

### PluginMigrationOp

Re-exports [PluginMigrationOp](generated.md#pluginmigrationop)

***

### PreparedStatement

Re-exports [PreparedStatement](db-execute.md#preparedstatement)

***

### PRODUCT\_API\_VERSION

Re-exports [PRODUCT_API_VERSION](abi.md#product_api_version)

***

### ProgressSink

Re-exports [ProgressSink](plugin.md#progresssink)

***

### PublishEvent

Re-exports [PublishEvent](env.md#publishevent)

***

### PutResult

Re-exports [PutResult](plugin.md#putresult)

***

### QueueBatch

Re-exports [QueueBatch](plugin.md#queuebatch)

***

### queueBatchResults

Re-exports [queueBatchResults](plugin.md#queuebatchresults)

***

### QueueBinding

Re-exports [QueueBinding](env.md#queuebinding)

***

### QueueMessage

Re-exports [QueueMessage](plugin.md#queuemessage)

***

### QueueOutcome

Re-exports [QueueOutcome](plugin.md#queueoutcome)

***

### QueueRetryOptions

Re-exports [QueueRetryOptions](plugin.md#queueretryoptions)

***

### QueueSendBatchEntry

Re-exports [QueueSendBatchEntry](env.md#queuesendbatchentry)

***

### QueueSendOptions

Re-exports [QueueSendOptions](env.md#queuesendoptions)

***

### rawStreamCeiling

Re-exports [rawStreamCeiling](README.md#rawstreamceiling)

***

### ReadOptions

Re-exports [ReadOptions](plugin.md#readoptions)

***

### ReadResult

Re-exports [ReadResult](plugin.md#readresult)

***

### RemoteLibraryEntrypoint

Re-exports [RemoteLibraryEntrypoint](plugin.md#remotelibraryentrypoint)

***

### requirePluginMigrationRegistration

Re-exports [requirePluginMigrationRegistration](README.md#requirepluginmigrationregistration)

***

### RetryOptions

Re-exports [RetryOptions](plugin.md#retryoptions)

***

### RetryToken

Re-exports [RetryToken](db-execute.md#retrytoken)

***

### runAdapterBatch

Re-exports [runAdapterBatch](README.md#runadapterbatch)

***

### schemaMigrationOp

Re-exports [schemaMigrationOp](plugin.md#schemamigrationop)

***

### SeriesEntry

Re-exports [SeriesEntry](generated.md#seriesentry)

***

### shouldSkipRemainingGuestWork

Re-exports [shouldSkipRemainingGuestWork](README.md#shouldskipremainingguestwork)

***

### Source

Re-exports [Source](plugin.md#source-2)

***

### StatementResult

Re-exports [StatementResult](db-execute.md#statementresult)

***

### statementResultToD1Result

Re-exports [statementResultToD1Result](db-execute.md#statementresulttod1result)

***

### StorageBinding

Re-exports [StorageBinding](env.md#storagebinding)

***

### StorageEntrypoint

Re-exports [StorageEntrypoint](plugin.md#storageentrypoint)

***

### StorefrontEntrypoint

Re-exports [StorefrontEntrypoint](plugin.md#storefrontentrypoint)

***

### STREAM\_FRAME\_TARGET\_BYTES

Re-exports [STREAM_FRAME_TARGET_BYTES](README.md#stream_frame_target_bytes)

***

### streamsResults

Re-exports [streamsResults](README.md#streamsresults)

***

### toBridgeJson

Re-exports [toBridgeJson](plugin.md#tobridgejson)

***

### TRIGGER\_FAMILIES

Re-exports [TRIGGER_FAMILIES](plugin.md#trigger_families)

***

### TypedDbStatement

Re-exports [TypedDbStatement](db-execute.md#typeddbstatement)

***

### WireQueueBatch

Re-exports [WireQueueBatch](plugin.md#wirequeuebatch)

***

### WorkKind

Re-exports [WorkKind](abi.md#workkind)

***

### wrapPluginFromBinding

Re-exports [wrapPluginFromBinding](plugin.md#wrappluginfrombinding)

***

### wrapPluginFromNative

Re-exports [wrapPluginFromNative](plugin.md#wrappluginfromnative)

***

### WriteChunksError

Re-exports [WriteChunksError](db-execute.md#writechunkserror)

***

### writeChunksOperationId

Re-exports [writeChunksOperationId](db-execute.md#writechunksoperationid)

***

### WriteChunksOptions

Re-exports [WriteChunksOptions](db-execute.md#writechunksoptions)

***

### WriteOptions

Re-exports [WriteOptions](plugin.md#writeoptions)

***

### WriteProgress

Re-exports [WriteProgress](db-execute.md#writeprogress)
