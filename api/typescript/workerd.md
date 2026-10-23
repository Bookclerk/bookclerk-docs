# workerd

## Classes

### InvokeError

**`Internal`**

Transport-level `/invoke` failure (never an author failure): unknown
interface/method, malformed message, bad capability table, or a missing
binding.

#### Extends

- [`PluginError`](index.md#pluginerror)

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

[`PluginError`](index.md#pluginerror).[`constructor`](index.md#constructor-10)

#### Properties

##### code

```ts
readonly code: string;
```

Known `PluginErrorCode` wire string, or `unknown`.

###### Inherited from

[`PluginError`](index.md#pluginerror).[`code`](index.md#code)

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

[`PluginError`](index.md#pluginerror).[`wireCode`](index.md#wirecode)

#### Methods

##### fromWire()

```ts
static fromWire(code, message): PluginError;
```

Construct a [PluginError](index.md#pluginerror) from a Cap'n Proto / JSON wire code.

Unknown codes become `unknown` on [PluginError.code](index.md#code) while
[PluginError.wireCode](index.md#wirecode) keeps the raw value.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `code` | `string` | Wire error code (known or unknown). |
| `message` | `string` | Operator-facing error text. |

###### Returns

[`PluginError`](index.md#pluginerror)

Typed plugin error.

###### Inherited from

[`PluginError`](index.md#pluginerror).[`fromWire`](index.md#fromwire)

## Interfaces

### CapDescriptor

**`Internal`**

One `X-Bookclerk-Caps` entry: capability index `i` of the message is
described by entry `i`.

#### Properties

##### binding?

```ts
optional binding?: string;
```

Binding name for `guestDatabase`.

##### id?

```ts
optional id?: string;
```

Isolate object id for `adapterSession`.

##### kind

```ts
kind: string;
```

`source` / `destination` / `progress` / `cancellation` / … / `adapterSession`.

##### token?

```ts
optional token?: string;
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

[`AuthorStub`](index.md#authorstub) \| `undefined`

##### bindContext()

```ts
bindContext(ctx): GrantedContext;
```

Bridge context → author-facing granted context.

###### Parameters

| Parameter | Type |
| ------ | ------ |
| `ctx` | `BridgeContext` \| `undefined` |

###### Returns

[`GrantedContext`](index.md#grantedcontext)

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
| `name` | [`EntrypointName`](index.md#entrypointname) |

###### Returns

[`NamedStub`](index.md#namedstub) \| `undefined`

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

## Variables

### MAX\_EVENT\_PAYLOAD\_BYTES

```ts
const MAX_EVENT_PAYLOAD_BYTES: 65536;
```

Maximum decoded size of a domain-event scalar payload (not a stream).

***

### TRIGGER\_FAMILIES

```ts
const TRIGGER_FAMILIES: Readonly<Record<"eventConsumer" | "queueConsumer" | "jobRunner", string>>;
```

Wire names of the trigger families the launcher may ask `openInvocation` about.

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
| `context` | `BridgeContext` \| `undefined` | Decoded bridge context, or `undefined` for `PluginWorker.*`. |
| `caps` | [`CapDescriptor`](#capdescriptor)[] | Request capability descriptors (`X-Bookclerk-Caps`). |
| `target` | `string` \| `null` | Isolate object id (`X-Bookclerk-Target`), or `null`. |
| `body` | `Uint8Array` | `$Params` message bytes. |
| `host` | [`InvokeHost`](#invokehost) | Adapter hooks. |

#### Returns

`Promise`\<[`InvokeReply`](#invokereply)\>

`$Results` bytes plus reply capability descriptors.

#### Throws

On transport-level failures (`400` / `404` / `500`).

***

### fromBridgeJson()

```ts
function fromBridgeJson(value): unknown;
```

Bridge JSON → typed ABI value: base64 `Data` fields (`payload`, `sha256`,
`credentials`) become `Uint8Array`; everything else is structurally
identical, so no per-struct codec is needed.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `value` | `unknown` | Bridge JSON value. |

#### Returns

`unknown`

Typed struct value.

## References

### AdapterDatabaseSession

Re-exports [AdapterDatabaseSession](index.md#adapterdatabasesession)

***

### AdapterEnv

Re-exports [AdapterEnv](index.md#adapterenv)

***

### allowedEntrypointFamilies

Re-exports [allowedEntrypointFamilies](index.md#allowedentrypointfamilies)

***

### AtomicTransport

Re-exports [AtomicTransport](index.md#atomictransport)

***

### BookclerkEntrypoint

Re-exports [BookclerkEntrypoint](index.md#bookclerkentrypoint)

***

### BookclerkEnv

Re-exports [BookclerkEnv](index.md#bookclerkenv)

***

### canonicalExecuteRequestHash

Re-exports [canonicalExecuteRequestHash](index.md#canonicalexecuterequesthash)

***

### Checkpoint

Re-exports [Checkpoint](index.md#checkpoint-4)

***

### cliArgs

Re-exports [cliArgs](index.md#cliargs)

***

### CliEntrypoint

Re-exports [CliEntrypoint](index.md#clientrypoint)

***

### CliInvokeParams

Re-exports [CliInvokeParams](index.md#cliinvokeparams)

***

### CliInvokeResult

Re-exports [CliInvokeResult](index.md#cliinvokeresult)

***

### CliSchema

Re-exports [CliSchema](index.md#clischema)

***

### CopyResult

Re-exports [CopyResult](index.md#copyresult)

***

### createDatabaseBinding

Re-exports [createDatabaseBinding](index.md#createdatabasebinding)

***

### D1ExecResult

Re-exports [D1ExecResult](index.md#d1execresult)

***

### D1Meta

Re-exports [D1Meta](index.md#d1meta)

***

### D1Result

Re-exports [D1Result](index.md#d1result)

***

### DatabaseAdapterEntrypoint

Re-exports [DatabaseAdapterEntrypoint](index.md#databaseadapterentrypoint)

***

### DatabaseBinding

Re-exports [DatabaseBinding](index.md#databasebinding)

***

### DatabaseBindingOptions

Re-exports [DatabaseBindingOptions](index.md#databasebindingoptions)

***

### dataMigrationOp

Re-exports [dataMigrationOp](index.md#datamigrationop)

***

### DbColumn

Re-exports [DbColumn](index.md#dbcolumn)

***

### DbResultSelection

Re-exports [DbResultSelection](index.md#dbresultselection)

***

### DbRow

Re-exports [DbRow](index.md#dbrow)

***

### DbStatementKind

Re-exports [DbStatementKind](index.md#dbstatementkind-1)

***

### DbTiming

Re-exports [DbTiming](index.md#dbtiming)

***

### DbType

Re-exports [DbType](index.md#dbtype-1)

***

### DbValue

Re-exports [DbValue](index.md#dbvalue)

***

### decodeDbValue

Re-exports [decodeDbValue](index.md#decodedbvalue)

***

### decodeExecuteRequest

Re-exports [decodeExecuteRequest](index.md#decodeexecuterequest)

***

### decodeExecuteResultReply

Re-exports [decodeExecuteResultReply](index.md#decodeexecuteresultreply)

***

### decodeExtensibleConfig

Re-exports [decodeExtensibleConfig](index.md#decodeextensibleconfig)

***

### DeliveredCheckpoint

Re-exports [DeliveredCheckpoint](index.md#deliveredcheckpoint)

***

### Destination

Re-exports [Destination](index.md#destination)

***

### DomainEvent

Re-exports [DomainEvent](index.md#domainevent)

***

### encodeDbValue

Re-exports [encodeDbValue](index.md#encodedbvalue)

***

### encodeExecuteRequest

Re-exports [encodeExecuteRequest](index.md#encodeexecuterequest)

***

### encodeExecuteResultReply

Re-exports [encodeExecuteResultReply](index.md#encodeexecuteresultreply)

***

### ENTRYPOINT\_BINDINGS

Re-exports [ENTRYPOINT_BINDINGS](index.md#entrypoint_bindings)

***

### ENTRYPOINT\_CLASSES

Re-exports [ENTRYPOINT_CLASSES](index.md#entrypoint_classes)

***

### EntrypointName

Re-exports [EntrypointName](index.md#entrypointname)

***

### EventBatch

Re-exports [EventBatch](index.md#eventbatch)

***

### eventBatchResults

Re-exports [eventBatchResults](index.md#eventbatchresults)

***

### EventMessage

Re-exports [EventMessage](index.md#eventmessage)

***

### EventOutcome

Re-exports [EventOutcome](index.md#eventoutcome)

***

### EventPublisherBinding

Re-exports [EventPublisherBinding](index.md#eventpublisherbinding)

***

### EventSuspendOptions

Re-exports [EventSuspendOptions](index.md#eventsuspendoptions)

***

### ExecuteReply

Re-exports [ExecuteReply](index.md#executereply)

***

### executeReplyToD1Results

Re-exports [executeReplyToD1Results](index.md#executereplytod1results)

***

### ExecuteRequest

Re-exports [ExecuteRequest](index.md#executerequest)

***

### ExtensibleConfig

Re-exports [ExtensibleConfig](index.md#extensibleconfig)

***

### FEATURE\_SCALAR\_LIMITS

Re-exports [FEATURE_SCALAR_LIMITS](index.md#feature_scalar_limits)

***

### FEATURE\_STORAGE\_COPY

Re-exports [FEATURE_STORAGE_COPY](index.md#feature_storage_copy)

***

### FEATURE\_STREAMS

Re-exports [FEATURE_STREAMS](index.md#feature_streams)

***

### GrantedContext

Re-exports [GrantedContext](index.md#grantedcontext)

***

### GrantedDatabaseTransport

Re-exports [GrantedDatabaseTransport](index.md#granteddatabasetransport)

***

### HealthOk

Re-exports [HealthOk](index.md#healthok)

***

### Instant

Re-exports [Instant](index.md#instant)

***

### Invocation

Re-exports [Invocation](index.md#invocation-11)

***

### invocationOf

Re-exports [invocationOf](index.md#invocationof)

***

### JobCheckpoint

Re-exports [JobCheckpoint](index.md#jobcheckpoint)

***

### JobCompletion

Re-exports [JobCompletion](index.md#jobcompletion)

***

### JobController

Re-exports [JobController](index.md#jobcontroller)

***

### JobInvocation

Re-exports [JobInvocation](index.md#jobinvocation)

***

### jobOutcomeFor

Re-exports [jobOutcomeFor](index.md#joboutcomefor)

***

### JobOutcomeRecord

Re-exports [JobOutcomeRecord](index.md#joboutcomerecord)

***

### JobRetryOptions

Re-exports [JobRetryOptions](index.md#jobretryoptions)

***

### JobRunnerContext

Re-exports [JobRunnerContext](index.md#jobrunnercontext)

***

### JobSuspendOptions

Re-exports [JobSuspendOptions](index.md#jobsuspendoptions)

***

### JsonObject

Re-exports [JsonObject](index.md#jsonobject)

***

### jsonPayload

Re-exports [jsonPayload](index.md#jsonpayload)

***

### KvListKeyBinding

Re-exports [KvListKeyBinding](index.md#kvlistkeybinding)

***

### KvListOptionsBinding

Re-exports [KvListOptionsBinding](index.md#kvlistoptionsbinding)

***

### KvListResultBinding

Re-exports [KvListResultBinding](index.md#kvlistresultbinding)

***

### KvNamespaceBinding

Re-exports [KvNamespaceBinding](index.md#kvnamespacebinding)

***

### KvPutOptionsBinding

Re-exports [KvPutOptionsBinding](index.md#kvputoptionsbinding)

***

### KvValueType

Re-exports [KvValueType](index.md#kvvaluetype)

***

### KvValueWithMetadata

Re-exports [KvValueWithMetadata](index.md#kvvaluewithmetadata)

***

### ListOptions

Re-exports [ListOptions](index.md#listoptions)

***

### ListPage

Re-exports [ListPage](index.md#listpage)

***

### MAX\_CHECKPOINT\_BYTES

Re-exports [MAX_CHECKPOINT_BYTES](index.md#max_checkpoint_bytes)

***

### MAX\_LIST\_PAGE

Re-exports [MAX_LIST_PAGE](index.md#max_list_page)

***

### MAX\_PLUGIN\_MIGRATION\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_OPS](index.md#max_plugin_migration_ops)

***

### MAX\_PLUGIN\_MIGRATION\_REGISTRATION\_BYTES

Re-exports [MAX_PLUGIN_MIGRATION_REGISTRATION_BYTES](index.md#max_plugin_migration_registration_bytes)

***

### MAX\_PLUGIN\_MIGRATION\_TOTAL\_OPS

Re-exports [MAX_PLUGIN_MIGRATION_TOTAL_OPS](index.md#max_plugin_migration_total_ops)

***

### MAX\_SCALAR\_BYTES

Re-exports [MAX_SCALAR_BYTES](index.md#max_scalar_bytes)

***

### MAX\_STREAM\_WINDOW\_BYTES

Re-exports [MAX_STREAM_WINDOW_BYTES](index.md#max_stream_window_bytes)

***

### NamedDatabase

Re-exports [NamedDatabase](index.md#nameddatabase)

***

### NamedEntrypoint

Re-exports [NamedEntrypoint](index.md#namedentrypoint)

***

### NamedKv

Re-exports [NamedKv](index.md#namedkv)

***

### NamedQueue

Re-exports [NamedQueue](index.md#namedqueue)

***

### ObjectInfo

Re-exports [ObjectInfo](index.md#objectinfo)

***

### ObjectMetadata

Re-exports [ObjectMetadata](index.md#objectmetadata)

***

### OidcClientTemplate

Re-exports [OidcClientTemplate](index.md#oidcclienttemplate)

***

### OidcEntrypoint

Re-exports [OidcEntrypoint](index.md#oidcentrypoint)

***

### parseDbValue

Re-exports [parseDbValue](index.md#parsedbvalue)

***

### PluginCapabilities

Re-exports [PluginCapabilities](index.md#plugincapabilities)

***

### PluginDescribe

Re-exports [PluginDescribe](index.md#plugindescribe)

***

### PluginError

Re-exports [PluginError](index.md#pluginerror)

***

### PluginMigration

Re-exports [PluginMigration](index.md#pluginmigration)

***

### PluginMigrationOp

Re-exports [PluginMigrationOp](index.md#pluginmigrationop)

***

### PreparedStatement

Re-exports [PreparedStatement](index.md#preparedstatement)

***

### PRODUCT\_API\_VERSION

Re-exports [PRODUCT_API_VERSION](index.md#product_api_version)

***

### ProgressSink

Re-exports [ProgressSink](index.md#progresssink)

***

### PublishEvent

Re-exports [PublishEvent](index.md#publishevent)

***

### PutResult

Re-exports [PutResult](index.md#putresult)

***

### QueueBatch

Re-exports [QueueBatch](index.md#queuebatch)

***

### queueBatchResults

Re-exports [queueBatchResults](index.md#queuebatchresults)

***

### QueueBinding

Re-exports [QueueBinding](index.md#queuebinding)

***

### QueueMessage

Re-exports [QueueMessage](index.md#queuemessage)

***

### QueueOutcome

Re-exports [QueueOutcome](index.md#queueoutcome)

***

### QueueRetryOptions

Re-exports [QueueRetryOptions](index.md#queueretryoptions)

***

### QueueSendBatchEntry

Re-exports [QueueSendBatchEntry](index.md#queuesendbatchentry)

***

### QueueSendOptions

Re-exports [QueueSendOptions](index.md#queuesendoptions)

***

### ReadOptions

Re-exports [ReadOptions](index.md#readoptions)

***

### ReadResult

Re-exports [ReadResult](index.md#readresult)

***

### RemoteLibraryEntrypoint

Re-exports [RemoteLibraryEntrypoint](index.md#remotelibraryentrypoint)

***

### requirePluginMigrationRegistration

Re-exports [requirePluginMigrationRegistration](index.md#requirepluginmigrationregistration)

***

### RetryOptions

Re-exports [RetryOptions](index.md#retryoptions)

***

### RetryToken

Re-exports [RetryToken](index.md#retrytoken)

***

### schemaMigrationOp

Re-exports [schemaMigrationOp](index.md#schemamigrationop)

***

### Source

Re-exports [Source](index.md#source-2)

***

### StatementResult

Re-exports [StatementResult](index.md#statementresult)

***

### statementResultToD1Result

Re-exports [statementResultToD1Result](index.md#statementresulttod1result)

***

### StorageBinding

Re-exports [StorageBinding](index.md#storagebinding)

***

### StorageEntrypoint

Re-exports [StorageEntrypoint](index.md#storageentrypoint)

***

### StorefrontEntrypoint

Re-exports [StorefrontEntrypoint](index.md#storefrontentrypoint)

***

### toBridgeJson

Re-exports [toBridgeJson](index.md#tobridgejson)

***

### TypedDbStatement

Re-exports [TypedDbStatement](index.md#typeddbstatement)

***

### WireQueueBatch

Re-exports [WireQueueBatch](index.md#wirequeuebatch)

***

### wrapPluginFromBinding

Re-exports [wrapPluginFromBinding](index.md#wrappluginfrombinding)

***

### wrapPluginFromNative

Re-exports [wrapPluginFromNative](index.md#wrappluginfromnative)

***

### WriteOptions

Re-exports [WriteOptions](index.md#writeoptions)
