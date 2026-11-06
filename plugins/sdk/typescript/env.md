# env

## Interfaces

### BookclerkEnv

Guest `env` bindings for `api_version = 3` manifests. Every host-provided
facility is a binding here; there is no `HOST` object. Bindings appear only
when `plugin.toml` declares them and the operator has consented.

Run `bookclerk-plugin types` to generate a precise `Env` for one manifest
(`bookclerk-configuration.d.ts`), then `extends BookclerkEntrypoint<Env>`.

#### Indexable

```ts
[binding: string]: unknown
```

Named `[[databases]]` bindings, `[[queues.producers]]` bindings, and any
renamed `binding = "..."` entries.

#### Properties

##### CONFIG?

```ts
CONFIG?: JsonObject;
```

`[vars]` plus operator settings, decoded from the granted `CONFIG` payload.

##### EVENTS?

```ts
EVENTS?: EventPublisherBinding;
```

`[[events.producers]]` outbox publisher.

##### KV?

```ts
KV?: KvNamespaceBinding;
```

`[[kv_namespaces]]` durable KV binding (default name).

##### OAUTH?

```ts
OAUTH?: unknown;
```

`[oauth]` loopback helper; surface reserved.

##### SECRETS?

```ts
SECRETS?: JsonObject;
```

`[secrets]` values, decoded from the granted `SECRETS` payload.

##### WORK\_FS?

```ts
WORK_FS?: StorageBinding;
```

`[work_fs]` object storage.

***

### EventPublisherBinding

`EVENTS` binding: outbox publisher, present when `[[events.producers]]` is
declared and granted.

#### Methods

##### publish()

```ts
publish(event): Promise<PublishOk>;
```

Publish one domain event through the host outbox.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `event` | [`PublishEvent`](#publishevent) | Event type, schema version, payload, dedup key. |

###### Returns

`Promise`\<[`PublishOk`](generated.md#publishok)\>

Outbox row identity (`duplicate` when the dedup key matched).

***

### KvListKeyBinding

One key on a [KvNamespaceBinding.list](#list) page.

#### Properties

##### expiration?

```ts
expiration?: number;
```

Absolute expiry in seconds since the Unix epoch, when set.

##### metadata?

```ts
metadata?: unknown;
```

Stored metadata, when set.

##### name

```ts
name: string;
```

Key name.

***

### KvListOptionsBinding

Page request for [KvNamespaceBinding.list](#list).

#### Properties

##### cursor?

```ts
cursor?: string;
```

Cursor from a previous page.

##### limit?

```ts
limit?: number;
```

Most keys to return (host default when omitted, at most `maxListPage`).

##### prefix?

```ts
prefix?: string;
```

Only keys starting with this prefix.

***

### KvListResultBinding

One page of [KvNamespaceBinding.list](#list).

#### Properties

##### cursor?

```ts
cursor?: string;
```

Cursor for the next page; absent when `list_complete`.

##### keys

```ts
keys: KvListKeyBinding[];
```

Keys in lexical order.

##### list\_complete

```ts
list_complete: boolean;
```

True when no further page exists.

***

### KvNamespaceBinding

`[[kv_namespaces]]` binding (`env.<BINDING>`, default `KV`): Workers
`KVNamespace`-shaped durable storage the host serves from the library
database, scoped to this plugin and the namespace the row names. Values
follow the instance across hosts.

#### Methods

##### delete()

```ts
delete(key): Promise<void>;
```

Delete one key; a missing key is not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to delete. |

###### Returns

`Promise`\<`void`\>

##### get()

###### Call Signature

```ts
get(key, type?): Promise<string | null>;
```

Read one key.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type?` | `"text"` | Decoding (`text` by default). |

###### Returns

`Promise`\<`string` \| `null`\>

The value, or `null` when absent or expired.

###### Call Signature

```ts
get<T>(key, type): Promise<T | null>;
```

Read one key and parse it as JSON.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-t"></a> `T` | `unknown` |

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type` | `"json"` | `json`. |

###### Returns

`Promise`\<`T` \| `null`\>

The parsed value, or `null` when absent or expired.

###### Call Signature

```ts
get(key, type): Promise<ArrayBuffer | null>;
```

Read one key as bytes.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key, at most `maxKvKeyBytes` UTF-8 bytes. |
| `type` | `"arrayBuffer"` | `arrayBuffer`. |

###### Returns

`Promise`\<`ArrayBuffer` \| `null`\>

The bytes, or `null` when absent or expired.

##### getWithMetadata()

###### Call Signature

```ts
getWithMetadata(key, type?): Promise<KvValueWithMetadata<string>>;
```

Read one key with its metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type?` | `"text"` | Decoding (`text` by default). |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`string`\>\>

Value and metadata; both `null` when absent.

###### Call Signature

```ts
getWithMetadata<T>(key, type): Promise<KvValueWithMetadata<T>>;
```

Read one key as JSON with its metadata.

###### Type Parameters

| Type Parameter | Default type |
| ------ | ------ |
| <a id="type-parameter-t-1"></a> `T` | `unknown` |

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type` | `"json"` | `json`. |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`T`\>\>

Parsed value and metadata; both `null` when absent.

###### Call Signature

```ts
getWithMetadata(key, type): Promise<KvValueWithMetadata<ArrayBuffer>>;
```

Read one key as bytes with its metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to read. |
| `type` | `"arrayBuffer"` | `arrayBuffer`. |

###### Returns

`Promise`\<[`KvValueWithMetadata`](#kvvaluewithmetadata)\<`ArrayBuffer`\>\>

Bytes and metadata; both `null` when absent.

##### list()

```ts
list(options?): Promise<KvListResultBinding>;
```

List keys in lexical order, one page at a time.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`KvListOptionsBinding`](#kvlistoptionsbinding) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`KvListResultBinding`](#kvlistresultbinding)\>

One page of keys.

##### put()

```ts
put(
   key, 
   value, 
   options?
): Promise<void>;
```

Write one key, replacing any previous value and metadata.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Key to write. |
| `value` | `string` \| `ArrayBuffer` \| `ArrayBufferView`\<`ArrayBufferLike`\> | String (stored as UTF-8) or bytes. |
| `options?` | [`KvPutOptionsBinding`](#kvputoptionsbinding) | Expiration and metadata. |

###### Returns

`Promise`\<`void`\>

***

### KvPutOptionsBinding

Options for [KvNamespaceBinding.put](#put) (Workers KV names).

#### Properties

##### expiration?

```ts
expiration?: number;
```

Absolute expiry in seconds since the Unix epoch.

##### expirationTtl?

```ts
expirationTtl?: number;
```

Relative expiry in seconds from the write; ignored when `expiration` is set.

##### metadata?

```ts
metadata?: unknown;
```

JSON-serializable metadata, at most `maxKvMetadataBytes` once encoded.

***

### KvValueWithMetadata

Value plus metadata from [KvNamespaceBinding.getWithMetadata](#getwithmetadata).

#### Type Parameters

| Type Parameter |
| ------ |
| <a id="type-parameter-t-2"></a> `T` |

#### Properties

##### metadata

```ts
metadata: unknown;
```

Stored metadata, or `null` when none.

##### value

```ts
value: T | null;
```

Decoded value, or `null` when the key is absent or expired.

***

### QueueBinding

`[[queues.producers]]` binding (`env.<BINDING>`): Workers `Queue`-shaped
producer into the queue the row names. The host assigns ids, stamps the
sending plugin as `source`, and bounds each body at `maxEventPayloadBytes`.

#### Methods

##### send()

```ts
send(body, options?): Promise<QueueSendOk>;
```

Send one message.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `body` | `unknown` | JSON value (default), string, or bytes per `contentType`. |
| `options?` | [`QueueSendOptions`](#queuesendoptions) | Encoding and delay. |

###### Returns

`Promise`\<[`QueueSendOk`](generated.md#queuesendok)\>

The outbox id the host assigned.

##### sendBatch()

```ts
sendBatch(messages, options?): Promise<QueueSendOk>;
```

Send up to `maxListPage` messages in order.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `messages` | readonly [`QueueSendBatchEntry`](#queuesendbatchentry)[] | Bodies with per-message options. |
| `options?` | `Pick`\<[`QueueSendOptions`](#queuesendoptions), `"delaySeconds"`\> | Default delay for entries that carry none. |

###### Returns

`Promise`\<[`QueueSendOk`](generated.md#queuesendok)\>

The outbox ids in send order.

***

### QueueSendBatchEntry

One entry of [QueueBinding.sendBatch](#sendbatch).

#### Extends

- [`QueueSendOptions`](#queuesendoptions)

#### Properties

##### body

```ts
body: unknown;
```

Message body (JSON value, string, or bytes per `contentType`).

##### contentType?

```ts
contentType?: "text" | "bytes" | "json";
```

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

###### Inherited from

[`QueueSendOptions`](#queuesendoptions).[`contentType`](#contenttype-1)

##### delaySeconds?

```ts
delaySeconds?: number;
```

Seconds before the message may be delivered; `0` means now.

###### Inherited from

[`QueueSendOptions`](#queuesendoptions).[`delaySeconds`](#delayseconds-1)

***

### QueueSendOptions

Per-send options for a [QueueBinding](#queuebinding).

#### Extended by

- [`QueueSendBatchEntry`](#queuesendbatchentry)

#### Properties

##### contentType?

```ts
contentType?: "text" | "bytes" | "json";
```

Body encoding: `json` (default; the body is JSON-encoded), `text`
(UTF-8 string), or `bytes` (raw `Uint8Array`).

##### delaySeconds?

```ts
delaySeconds?: number;
```

Seconds before the message may be delivered; `0` means now.

***

### StorageBinding

`WORK_FS` binding: host-granted object storage for durable plugin files.
Same surface as the `storage` entrypoint the host calls on storage plugins.

#### Methods

##### delete()

```ts
delete(key): Promise<void>;
```

Remove an object; missing keys are not an error.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |

###### Returns

`Promise`\<`void`\>

Resolves when the delete is durable.

##### get()

```ts
get(key, options?): Promise<{
  body: ReadableStream<Uint8Array<ArrayBufferLike>>;
  meta: {
     key: string;
     size: number;
  };
}>;
```

Stream an object (optionally a byte range).

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |
| `options?` | \{ `range?`: \{ `length?`: `number`; `offset`: `number`; \}; \} | Optional byte range. |
| `options.range?` | \{ `length?`: `number`; `offset`: `number`; \} | - |
| `options.range.length?` | `number` | - |
| `options.range.offset` | `number` | - |

###### Returns

`Promise`\<\{
  `body`: `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\>;
  `meta`: \{
     `key`: `string`;
     `size`: `number`;
  \};
\}\>

Metadata and the body stream.

##### head()

```ts
head(key): Promise<
  | {
  contentType?: string;
  etag?: string;
  key: string;
  size: number;
}
| null>;
```

Object metadata without the body.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |

###### Returns

`Promise`\<
  \| \{
  `contentType?`: `string`;
  `etag?`: `string`;
  `key`: `string`;
  `size`: `number`;
\}
  \| `null`\>

Metadata, or `null` when the key does not exist.

##### list()

```ts
list(options): Promise<{
  nextCursor?: string;
  objects: object[];
}>;
```

One page of keys under `prefix`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options` | \{ `cursor?`: `string`; `limit?`: `number`; `prefix?`: `string`; \} | Prefix, continuation cursor, and page size. |
| `options.cursor?` | `string` | - |
| `options.limit?` | `number` | - |
| `options.prefix?` | `string` | - |

###### Returns

`Promise`\<\{
  `nextCursor?`: `string`;
  `objects`: `object`[];
\}\>

Objects plus the cursor for the next page, when any.

##### put()

```ts
put(
   key, 
   body, 
   options?
): Promise<{
  bytesWritten: number;
  key: string;
}>;
```

Write an object from a stream.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `key` | `string` | Object key. |
| `body` | `ReadableStream`\<`Uint8Array`\<`ArrayBufferLike`\>\> | Body bytes. |
| `options?` | \{ `contentLength?`: `number`; `contentType?`: `string`; \} | Content type and length hints. |
| `options.contentLength?` | `number` | - |
| `options.contentType?` | `string` | - |

###### Returns

`Promise`\<\{
  `bytesWritten`: `number`;
  `key`: `string`;
\}\>

Key and bytes written.

## Type Aliases

### JsonObject

```ts
type JsonObject = Record<string, unknown>;
```

Plain JSON object (string keys, JSON values).

***

### KvValueType

```ts
type KvValueType = "text" | "json" | "arrayBuffer";
```

How [KvNamespaceBinding.get](#get) decodes a stored value.

***

### PublishEvent

```ts
type PublishEvent = Pick<PluginEvent, "eventType"> & Partial<Omit<PluginEvent, "eventType" | "payload">> & object;
```

Event a guest publishes; the host stamps id, `source`, and `accountId`.

#### Type Declaration

| Name | Type | Description |
| ------ | ------ | ------ |
| `payload?` | `Uint8Array` \| [`JsonObject`](#jsonobject) \| `unknown`[] \| `string` | Payload bytes or a JSON-serializable value encoded as UTF-8 JSON. |
