# testkit

Contract conformance test kit: a plugin database binding that behaves
like the one a deployed host grants.

A plain in-memory fake accepts anything. On a host, `env.DB` reaches an
isolated unit through the adapter the operator deployed: SQL is scoped to
the plugin's own tables, every batch is one transaction held to the
adapter's advertised caps, and a result past a cap is a typed
`limit_exceeded` error. [createTestDatabase](#createtestdatabase) gives a Vitest or
`node:test` suite that same binding, backed by the `bookclerk-db-testkit`
binary from the Bookclerk workspace under one of three capability
profiles:

- `first-party`: the shipped sqlite and Postgres adapters (256 KiB
  results, streamed reads).
- `d1`: the D1 adapter (smaller payload and statement limits).
- `minimum`: the smallest advertisement a host connects to (4,096-byte
  results, 1,024-byte cells, one row per statement, no streamed reads).

Run plugin database code under all three ([TEST\_DATABASE\_PROFILES](#test_database_profiles)):
what works on the shipped adapters has to work when an adapter
advertises only the minimums.

```ts
import { createTestDatabase, TEST_DATABASE_PROFILES } from "@bookclerk/plugin-sdk/testkit";

for (const profile of TEST_DATABASE_PROFILES) {
  const db = await createTestDatabase({ profile, migrations: MIGRATIONS });
  try {
    await db.binding.prepare("INSERT INTO notes (id, body) VALUES (?, ?)").bind(1, "hi").run();
    const { results } = await db.binding.prepare("SELECT body FROM notes").all();
  } finally {
    await db.close();
  }
}
```

The binary is found through the `BOOKCLERK_DB_TESTKIT` environment
variable or `options.binary`, else as `bookclerk-db-testkit` on `PATH`
(`cargo build -p bookclerk-db-testkit` in the workspace builds it).
`bookclerk-plugin test` (`cargo plugin -- test <dir>` in the workspace)
runs a plugin's suite once per profile, setting both that variable and
`BOOKCLERK_DB_TESTKIT_PROFILE`, which is the default `profile`. This
module runs under Node, not inside an isolate.

## Interfaces

### TestDatabase

A served test binding and the process behind it.

#### Properties

##### binding

```ts
readonly binding: DatabaseBinding;
```

The D1-shaped binding, as `env.<BINDING>` on a host.

##### profile

```ts
readonly profile: "first-party" | "d1" | "minimum";
```

The profile being enforced.

#### Methods

##### close()

```ts
close(): Promise<void>;
```

Stops the kit process; the in-memory unit goes with it.

###### Returns

`Promise`\<`void`\>

Once the process has exited.

##### limits()

```ts
limits(): Promise<DbCapabilities>;
```

The profile's advertisement: the caps every batch is held to.

###### Returns

`Promise`\<[`DbCapabilities`](generated.md#dbcapabilities)\>

The capability advertisement, as `binding.limits()` reports it.

***

### TestDatabaseOptions

Options for [createTestDatabase](#createtestdatabase).

#### Properties

##### binary?

```ts
binary?: string;
```

Path of the `bookclerk-db-testkit` binary; see the module docs for the default.

##### migrations?

```ts
migrations?: PluginMigration[];
```

The plugin's `databaseMigrations` registration, applied before the
binding is handed out the way the host applies it at enrollment.

##### profile?

```ts
profile?: "first-party" | "d1" | "minimum";
```

Capability profile to enforce. Defaults to `BOOKCLERK_DB_TESTKIT_PROFILE`
when `bookclerk-plugin test` sets it, else `minimum`, the strictest.

***

### TestKv

A [KvNamespaceBinding](env.md#kvnamespacebinding) held in memory, with test controls.

#### Extends

- [`KvNamespaceBinding`](env.md#kvnamespacebinding)

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

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`delete`](env.md#delete)

##### failNext()

```ts
failNext(op, error): void;
```

Make the next call of `op` reject with `error` instead of running.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `op` | [`TestKvOperation`](#testkvoperation) | The operation. |
| `error` | [`PluginError`](README.md#pluginerror) | What it rejects with. |

###### Returns

`void`

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

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`get`](env.md#get)

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

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`get`](env.md#get)

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

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`get`](env.md#get)

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

`Promise`\<[`KvValueWithMetadata`](env.md#kvvaluewithmetadata)\<`string`\>\>

Value and metadata; both `null` when absent.

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`getWithMetadata`](env.md#getwithmetadata)

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

`Promise`\<[`KvValueWithMetadata`](env.md#kvvaluewithmetadata)\<`T`\>\>

Parsed value and metadata; both `null` when absent.

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`getWithMetadata`](env.md#getwithmetadata)

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

`Promise`\<[`KvValueWithMetadata`](env.md#kvvaluewithmetadata)\<`ArrayBuffer`\>\>

Bytes and metadata; both `null` when absent.

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`getWithMetadata`](env.md#getwithmetadata)

##### list()

```ts
list(options?): Promise<KvListResultBinding>;
```

List keys in lexical order, one page at a time.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`KvListOptionsBinding`](env.md#kvlistoptionsbinding) | Prefix, cursor, and page size. |

###### Returns

`Promise`\<[`KvListResultBinding`](env.md#kvlistresultbinding)\>

One page of keys.

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`list`](env.md#list)

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
| `options?` | [`KvPutOptionsBinding`](env.md#kvputoptionsbinding) | Expiration and metadata. |

###### Returns

`Promise`\<`void`\>

###### Inherited from

[`KvNamespaceBinding`](env.md#kvnamespacebinding).[`put`](env.md#put)

##### snapshot()

```ts
snapshot(): Map<string, string>;
```

Live keys and their values as text, for assertions.

###### Returns

`Map`\<`string`, `string`\>

Key to UTF-8 value, in key order.

***

### TestKvOptions

Options for [createTestKv](#createtestkv).

#### Properties

##### maxKeys?

```ts
maxKeys?: number;
```

The namespace's key-count limit; a new key past it is `forbidden`.

##### maxValueBytes?

```ts
maxValueBytes?: number;
```

The namespace's `max_value_bytes` (default and cap `MAX_KV_VALUE_BYTES`).

##### now?

```ts
now?: () => number;
```

Clock in milliseconds since the Unix epoch (default `Date.now`).

###### Returns

`number`

***

### TestWebhooks

A [WebhooksBinding](env.md#webhooksbinding) held in memory, with test controls.

#### Extends

- [`WebhooksBinding`](env.md#webhooksbinding)

#### Methods

##### ack()

```ts
ack(name, leases): Promise<void>;
```

Finish leased requests: `acks` are deleted, `retries` are offered again
on the next pull.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |
| `leases` | \{ `acks?`: `string`[]; `retries?`: `string`[]; \} | Lease ids to acknowledge and to retry. |
| `leases.acks?` | `string`[] | - |
| `leases.retries?` | `string`[] | - |

###### Returns

`Promise`\<`void`\>

Resolves when the host recorded both lists.

###### Inherited from

[`WebhooksBinding`](env.md#webhooksbinding).[`ack`](env.md#ack)

##### acked()

```ts
acked(name): string[];
```

Lease ids acknowledged for `name`, in order.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`string`[]

The ids.

##### deliver()

```ts
deliver(
   name, 
   body, 
   headers?
): void;
```

Queue one inbound request, as if the outside service sent it.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |
| `body` | `string` \| `Uint8Array`\<`ArrayBufferLike`\> | Body text or bytes. |
| `headers?` | `Record`\<`string`, `string`\> | Request headers. |

###### Returns

`void`

##### markMissed()

```ts
markMissed(name): void;
```

Make the next pull of `name` report `missed`, as when requests expired.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`void`

##### pull()

```ts
pull(name, options?): Promise<WebhookPullResult>;
```

Lease waiting requests for one webhook, oldest first.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |
| `options?` | [`WebhookPullOptions`](env.md#webhookpulloptions) | Batch size and lease length. |

###### Returns

`Promise`\<[`WebhookPullResult`](env.md#webhookpullresult)\>

The leased requests and whether any were missed.

###### Inherited from

[`WebhooksBinding`](env.md#webhooksbinding).[`pull`](env.md#pull)

##### retried()

```ts
retried(name): string[];
```

Lease ids released for retry for `name`, in order.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`string`[]

The ids.

##### url()

```ts
url(name): Promise<string | null>;
```

The URL to register with the outside service for one webhook.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`Promise`\<`string` \| `null`\>

The URL, or `null` when the library cannot be reached from the
  Internet or webhooks are turned off; use the fallback then.

###### Inherited from

[`WebhooksBinding`](env.md#webhooksbinding).[`url`](env.md#url-1)

##### waiting()

```ts
waiting(name): number;
```

Requests waiting (not leased) for `name`.

###### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `name` | `string` | `[[webhooks]]` name. |

###### Returns

`number`

The count.

***

### TestWebhooksOptions

Options for [createTestWebhooks](#createtestwebhooks).

#### Properties

##### baseUrl?

```ts
baseUrl?: string | null;
```

What `url(name)` returns: a base the name is appended to (default
`https://library.example/hooks/`), or `null` for a library the Internet
cannot reach.

## Type Aliases

### TestDatabaseProfile

```ts
type TestDatabaseProfile = typeof TEST_DATABASE_PROFILES[number];
```

One of [TEST\_DATABASE\_PROFILES](#test_database_profiles).

***

### TestKvOperation

```ts
type TestKvOperation = "get" | "put" | "delete" | "list";
```

Operations a test can make fail once with [TestKv.failNext](#failnext).

## Variables

### TEST\_DATABASE\_PROFILES

```ts
const TEST_DATABASE_PROFILES: readonly ["first-party", "d1", "minimum"];
```

The capability profiles the kit enforces, in the order conformance runs report them.

## Functions

### createTestDatabase()

```ts
function createTestDatabase(options?): Promise<TestDatabase>;
```

Spawns the kit under `options.profile` with `options.migrations` applied
and returns the served binding.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`TestDatabaseOptions`](#testdatabaseoptions) | Profile, registration and binary location. |

#### Returns

`Promise`\<[`TestDatabase`](#testdatabase)\>

The binding and its process; call `close()` when the test ends.

#### Throws

Error when the binary cannot be started, refuses the profile or
  the registration (the kit's message is included), or exits early.

***

### createTestKv()

```ts
function createTestKv(options?): TestKv;
```

Creates an in-memory KV namespace that applies the host's limits.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`TestKvOptions`](#testkvoptions) | Namespace limits and clock. |

#### Returns

[`TestKv`](#testkv)

The binding.

***

### createTestWebhooks()

```ts
function createTestWebhooks(options?): TestWebhooks;
```

Creates an in-memory `WEBHOOKS` binding that applies the host's limits.

Leased requests stay leased until acknowledged or retried; lease expiry
is not modelled, so release a lease with `ack(name, { retries })`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `options?` | [`TestWebhooksOptions`](#testwebhooksoptions) | The URL base. |

#### Returns

[`TestWebhooks`](#testwebhooks)

The binding.

***

### defaultProfile()

```ts
function defaultProfile(option): "first-party" | "d1" | "minimum";
```

The profile a test runs under: the option, else the one
`bookclerk-plugin test` names in `BOOKCLERK_DB_TESTKIT_PROFILE`, else
`minimum`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `option` | `"first-party"` \| `"d1"` \| `"minimum"` \| `undefined` | The caller's `profile` option. |

#### Returns

`"first-party"` \| `"d1"` \| `"minimum"`

The profile name, unchecked.

***

### kitMigrations()

```ts
function kitMigrations(migrations): object[];
```

Spells a `databaseMigrations` registration the way the kit reads it.

A plugin may return each operation typed (`{ kind: "schema", value }`,
the [PluginMigration](generated.md#pluginmigration) shape) or in the short form the
`schemaMigrationOp` / `dataMigrationOp` helpers build (`{ schema }`); the
host accepts both, so the kit is handed the one form it parses. The
registration is first held to the same resource bounds the isolate SDK
applies before the host sees it.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `migrations` | readonly [`PluginMigration`](generated.md#pluginmigration)[] | The registration, in either form. |

#### Returns

`object`[]

The same migrations with every operation as `{ schema }` or `{ data }`.

#### Throws

Error naming the migration when an operation is neither form, or
  with code `payload_too_large` when the registration passes an ABI bound.

***

### testkitBinary()

```ts
function testkitBinary(binary?): string;
```

Picks the kit binary: the option, then `BOOKCLERK_DB_TESTKIT`, then the
bare name on `PATH`.

#### Parameters

| Parameter | Type | Description |
| ------ | ------ | ------ |
| `binary?` | `string` | Explicit path, when given. |

#### Returns

`string`

The command to spawn.
