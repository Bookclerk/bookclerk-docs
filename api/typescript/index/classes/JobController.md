[**@bookclerk/plugin-sdk**](../../README.md)

***

[@bookclerk/plugin-sdk](../../modules.md) / [index](../README.md) / JobController

# Class: JobController

Defined in: [plugin.ts:1243](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1243)

Everything one job invocation may touch: the durable envelope, `input` /
`output` streams, `progress()`, `signal`, and the terminal-outcome recorders
`suspend()` / `retryLater()`. Returning normally without a recorded outcome
completes the job; throwing rejects it (`unavailable` /
`deadline_exceeded` errors are retryable, `cancelled` is cancelled).

## Constructors

### Constructor

> **new JobController**(`invocation`, `granted?`): `JobController`

Defined in: [plugin.ts:1281](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1281)

#### Parameters

##### invocation

`Partial`\<[`JobInvocation`](../interfaces/JobInvocation.md)\> \| `undefined`

##### granted?

[`GrantedJobCapabilities`](../interfaces/GrantedJobCapabilities.md)

#### Returns

`JobController`

## Properties

### attempt

> `readonly` **attempt**: `number`

Defined in: [plugin.ts:1261](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1261)

Failure retry counter, starting at 1.

***

### causationId

> `readonly` **causationId**: `string`

Defined in: [plugin.ts:1267](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1267)

Id of the event or command that caused this job.

***

### checkpoint

> `readonly` **checkpoint**: [`DeliveredCheckpoint`](../interfaces/DeliveredCheckpoint.md) \| `null`

Defined in: [plugin.ts:1273](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1273)

Checkpoint from the prior `suspend()`, or `null`.

***

### correlationId

> `readonly` **correlationId**: `string`

Defined in: [plugin.ts:1265](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1265)

Trace correlation id.

***

### deadlineUnixMs

> `readonly` **deadlineUnixMs**: `number`

Defined in: [plugin.ts:1263](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1263)

Deadline hint (Unix ms); the host fence is authoritative.

***

### id

> `readonly` **id**: `string`

Defined in: [plugin.ts:1251](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1251)

Unique id of this invocation attempt.

***

### idempotencyKey

> `readonly` **idempotencyKey**: `string`

Defined in: [plugin.ts:1259](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1259)

Caller idempotency key.

***

### input

> `readonly` **input**: [`Source`](Source.md) \| `null`

Defined in: [plugin.ts:1275](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1275)

Granted byte source, when the job has input.

***

### invocation

> `readonly` **invocation**: `Readonly`\<`Partial`\<[`JobInvocation`](../interfaces/JobInvocation.md)\>\>

Defined in: [plugin.ts:1249](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1249)

Full durable envelope as delivered.

***

### invocationSequence

> `readonly` **invocationSequence**: `number`

Defined in: [plugin.ts:1269](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1269)

Resume ordinal.

***

### output

> `readonly` **output**: [`Destination`](Destination.md) \| `null`

Defined in: [plugin.ts:1277](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1277)

Granted object store, when the job has output.

***

### payloadJson

> `readonly` **payloadJson**: `string`

Defined in: [plugin.ts:1255](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1255)

Command payload JSON text.

***

### payloadSchemaVersion

> `readonly` **payloadSchemaVersion**: `number`

Defined in: [plugin.ts:1257](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1257)

Schema version of [JobController.payloadJson](#payloadjson).

***

### signal

> `readonly` **signal**: `AbortSignal`

Defined in: [plugin.ts:1279](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1279)

Aborts when the host cancels the invocation.

***

### stepId

> `readonly` **stepId**: `string`

Defined in: [plugin.ts:1271](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1271)

Step id within a multi-step command; empty when single-step.

***

### type

> `readonly` **type**: `string`

Defined in: [plugin.ts:1253](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1253)

Command type the handler dispatches on.

## Accessors

### result

#### Get Signature

> **get** **result**(): [`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md) \| `null`

Defined in: [plugin.ts:1346](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1346)

Recorded terminal outcome.

##### Returns

[`JobOutcomeRecord`](../type-aliases/JobOutcomeRecord.md) \| `null`

The outcome, or `null` while the job is still running.

## Methods

### cancel()

> **cancel**(): `void`

Defined in: [plugin.ts:1386](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1386)

Host-side cancellation observed (adapter-internal).

#### Returns

`void`

***

### json()

> **json**\<`T`\>(): `T`

Defined in: [plugin.ts:1325](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1325)

Decode [JobController.payloadJson](#payloadjson).

#### Type Parameters

##### T

`T` = [`JsonObject`](../type-aliases/JsonObject.md)

#### Returns

`T`

Parsed payload (`{}` when empty).

***

### progress()

> **progress**(`percent`, `message?`): `Promise`\<`void`\>

Defined in: [plugin.ts:1336](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1336)

Report `percent` in `0..=100` with an operator-facing `message`.

#### Parameters

##### percent

`number`

Completion percent.

##### message?

`string`

Operator-facing status.

#### Returns

`Promise`\<`void`\>

Resolves when the host records progress.

***

### retryLater()

> **retryLater**(`options?`): `void`

Defined in: [plugin.ts:1376](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1376)

Give up this attempt and let the host retry at `retryAt` (or its default).

#### Parameters

##### options?

[`JobRetryOptions`](../interfaces/JobRetryOptions.md)

Retry time and reason.

#### Returns

`void`

***

### suspend()

> **suspend**(`options?`): `void`

Defined in: [plugin.ts:1359](https://github.com/Bookclerk/bookclerk/blob/cfd2c29b7936bb6a3f313f9e38051ad5afb4ab49/packages/plugin-sdk/src/plugin.ts#L1359)

Release with a bounded checkpoint; the host resumes at `wakeAt`.

#### Parameters

##### options?

[`JobSuspendOptions`](../interfaces/JobSuspendOptions.md)

Checkpoint and wake time.

#### Returns

`void`
